## Exercício 01: Gerenciamento de Usuários, Grupos e Permissões

### Solução:

```bash
# 1. Crie um grupo chamado projetodelta.
sudo groupadd projetodelta

# 2. Crie dois usuários: devuser1 e devuser2. Ambos devem ter a senha redhat123.
sudo useradd devuser1
echo "redhat123" | sudo passwd --stdin devuser1
sudo useradd devuser2
echo "redhat123" | sudo passwd --stdin devuser2

# 3. Adicione devuser1 e devuser2 ao grupo projetodelta como grupo secundário.
sudo usermod -aG projetodelta devuser1
sudo usermod -aG projetodelta devuser2

# Verificação:
id devuser1
id devuser2

# 4. Crie um diretório /srv/projetodelta.
sudo mkdir /srv/projetodelta

# 5. Defina o proprietário do diretório /srv/projetodelta para root e o grupo para projetodelta.
sudo chown root:projetodelta /srv/projetodelta

# 6. Defina as permissões para /srv/projetodelta:
# root: rwx (7)
# projetodelta: rwx (7)
# outros: --- (0)
# Para este cenário, o grupo precisa de 'x' para entrar no diretório.
# Se o requisito fosse apenas leitura e escrita de arquivos, e não navegar para dentro,
# então seria 6. Mas para criar/modificar arquivos, 'x' é necessário no diretório.
# Vamos assumir que "acesso de leitura e escrita" implica na capacidade de criar e modificar arquivos, o que requer 'x'.
sudo chmod 2770 /srv/projetodelta # O '2' é para o bit SGID (set group ID)

# 7. Configure o diretório /srv/projetodelta para que todos os arquivos e subdiretórios
# criados dentro dele herdem o grupo projetodelta automaticamente.
# Isso já foi feito com o '2' no chmod (bit SGID). O '2' representa o bit SGID.
# Para verificar, você pode fazer:
ls -ld /srv/projetodelta
# A saída deve mostrar algo como drwxrws--- (o 's' no lugar do 'x' para o grupo indica SGID)

# 8. Como devuser1, crie um arquivo /srv/projetodelta/relatorio_dev1.txt e adicione algum texto.
# (Troque para o usuário devuser1)
su - devuser1 <<'EOF'
echo "Este é o relatório do dev1." > /srv/projetodelta/relatorio_dev1.txt
EOF
# Volte para o usuário root/sudo
# exit # Descomente se você não estiver usando EOF aqui para sair do subshell

# 9. Verifique se o arquivo relatorio_dev1.txt tem o grupo projetodelta.
ls -l /srv/projetodelta/relatorio_dev1.txt
# A saída deve mostrar que o grupo é 'projetodelta'.
```

-----

## Exercício 02: Gerenciamento de Armazenamento Local (LVM)

### Solução:

```bash
# 1. Crie um Volume Físico (PV) no novo disco /dev/sdb.
sudo pvcreate /dev/sdb

# Verificação:
sudo pvs

# 2. Crie um Grupo de Volumes (VG) chamado vg_data usando o PV recém-criado.
sudo vgcreate vg_data /dev/sdb

# Verificação:
sudo vgs

# 3. Crie um Volume Lógico (LV) chamado lv_web_content de 2GB dentro de vg_data.
sudo lvcreate -n lv_web_content -L 2G vg_data

# Verificação:
sudo lvs

# 4. Formate lv_web_content com o sistema de arquivos XFS.
sudo mkfs.xfs /dev/vg_data/lv_web_content

# 5. Monte lv_web_content em /var/www/html/site.
sudo mkdir -p /var/www/html/site
sudo mount /dev/vg_data/lv_web_content /var/www/html/site

# Verificação:
df -h /var/www/html/site

# 6. Configure o sistema para montar lv_web_content automaticamente no boot, usando o UUID.
# Primeiro, obtenha o UUID:
UUID=$(sudo blkid -s UUID -o value /dev/vg_data/lv_web_content)
echo "UUID do LV: $UUID"

# Adicione a entrada ao /etc/fstab:
# Abra o /etc/fstab com um editor de texto (vi/nano):
sudo vi /etc/fstab
# Adicione a seguinte linha:
# UUID=[COLE_SEU_UUID_AQUI] /var/www/html/site xfs defaults 0 0

# Teste o fstab para garantir que não há erros antes de reiniciar:
sudo mount -a
# Se não houver erros, nenhum output será exibido.

# 7. Estenda lv_web_content em 1GB.
sudo lvextend -L +1G /dev/vg_data/lv_web_content

# Verificação:
sudo lvs

# 8. Estenda o sistema de arquivos XFS para usar o novo espaço.
sudo xfs_growfs /var/www/html/site

# Verificação final:
df -h /var/www/html/site
```

-----

## Exercício 03: Configuração de Rede e FirewallD

### Solução:

```bash
# 1. Configure a interface de rede principal com IP estático.
# (Substitua 'enp0s3' pelo nome da sua interface, ex: ens192, eth0)
# A forma mais robusta é usando nmcli:
sudo nmcli connection modify enp0s3 ipv4.method manual ipv4.addresses 192.168.xxx.10/24 autoconnect yes connection.zone public

# Para o hostname, edite o arquivo /etc/hostname:
sudo hostnamectl set-hostname servidorweb.example.com

# 2. Reinicie o serviço de rede.
sudo nmcli connection up enp0s3
# Ou: sudo systemctl restart NetworkManager

# 3. Verifique se o hostname e as configurações de IP foram aplicados.
hostname
ip a show enp0s3 # Ou o nome da sua interface
ping -c 3 google.com

# 4. Abra a porta 80 (HTTP) e 443 (HTTPS) no FirewallD permanentemente.
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# 5. Recarregue o FirewallD para aplicar as regras imediatamente.
sudo firewall-cmd --reload

# 6. Verifique se as portas estão abertas.
sudo firewall-cmd --list-all

# 7. Instale o pacote httpd (Apache) e configure-o para iniciar automaticamente no boot.
sudo dnf install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Verificação:
sudo systemctl status httpd

# 8. Crie um arquivo index.html simples em /var/www/html e verifique o acesso via curl localhost.
echo "<h1>Bem-vindo ao Servidor Web de Teste!</h1>" | sudo tee /var/www/html/index.html

# Verificação:
curl localhost
```

-----

## Exercício 04: Agendamento de Tarefas (Cron) e Monitoramento de Logs

### Solução:

```bash
# 1. Crie um script simples monitora_memoria.sh em /usr/local/bin.
sudo vi /usr/local/bin/monitora_memoria.sh
# Adicione o seguinte conteúdo:
#!/bin/bash
DATA_HORA=$(date +"%Y-%m-%d %H:%M:%S")
MEMORIA_LIVRE=$(free -h | grep Mem | awk '{print $7}')
echo "$DATA_HORA - Memória Livre: $MEMORIA_LIVRE" >> /var/log/memoria.log

# Dê permissão de execução ao script:
sudo chmod +x /usr/local/bin/monitora_memoria.sh

# 2. Configure uma tarefa Cron para que o script seja executado todos os dias à 01:00 AM.
sudo crontab -e
# Adicione a seguinte linha no final do arquivo:
0 1 * * * /usr/local/bin/monitora_memoria.sh

# 3. Verifique se a tarefa Cron foi agendada corretamente para o usuário root.
sudo crontab -l

# 4. Configure a rotação de logs para /var/log/memoria.log usando logrotate.
sudo vi /etc/logrotate.d/memoria_log
# Adicione o seguinte conteúdo:
/var/log/memoria.log {
    weekly
    rotate 4
    compress
    notifempty
    missingok
}

# Você pode testar a configuração do logrotate manualmente (sem forçar a rotação real ainda):
sudo logrotate -d /etc/logrotate.d/memoria_log

# 5. Use journalctl para verificar os logs do serviço crond.
# (Você pode precisar esperar a hora agendada ou forçar a execução do cron para ver o log)
# Para forçar a execução (apenas para teste, não faça isso em produção):
# sudo /usr/local/bin/monitora_memoria.sh
# E depois verifique o log:
cat /var/log/memoria.log

# Para verificar no journalctl (mostrará se o cron iniciou o script, mas não o output do script):
sudo journalctl -u crond.service --since "yesterday"
```

-----

## Exercício 05: Gerenciamento de Serviços (Systemd) e Processos

### Solução:

```bash
# 1. Instale o pacote nginx.
sudo dnf install -y nginx

# 2. Habilite o serviço nginx para iniciar no boot.
sudo systemctl enable nginx

# 3. Inicie o serviço nginx.
sudo systemctl start nginx

# 4. Verifique o status do serviço nginx e garanta que ele está ativo e rodando.
sudo systemctl status nginx
# A saída deve mostrar "Active: active (running)"

# 5. Pare o serviço nginx.
sudo systemctl stop nginx

# Verificação:
sudo systemctl status nginx
# A saída deve mostrar "Active: inactive (dead)"

# 6. Reinicie o serviço nginx.
sudo systemctl restart nginx

# Verificação:
sudo systemctl status nginx
# A saída deve mostrar "Active: active (running)"

# 7. Identifique o processo principal do nginx (o processo pai) e veja seu PID.
# Use 'ps aux' para listar processos e grep por nginx.
# O processo principal geralmente tem PID baixo e é o pai de outros processos nginx.
PID_NGINX=$(ps aux | grep "[n]ginx: master process" | awk '{print $2}')
echo "PID do processo principal do Nginx: $PID_NGINX"

# 8. Altere a prioridade (nice value) do processo principal do nginx para -5.
# O comando 'renice' é usado para alterar o nice value de um processo em execução.
# -n é para o novo nice value, -p é para o PID.
sudo renice -n -5 -p $PID_NGINX

# 9. Verifique a nova prioridade do processo.
# Use 'ps -o pid,ni,comm -p <PID>' para ver o nice value (NI).
ps -o pid,ni,comm -p $PID_NGINX
# A coluna 'NI' deve mostrar -5.

# 10. Desabilite o serviço nginx para que ele não inicie automaticamente no boot.
sudo systemctl disable nginx

# Verificação:
sudo systemctl is-enabled nginx
# A saída deve ser "disabled"
```

-----

## Exercício 06: Gerenciamento de Armazenamento Local - Swap e Redimensionamento

### Solução:

```bash
# PRÉ-REQUISITO: Certifique-se de ter um VG existente (ex: vg_data do exercício anterior)
# Se não tiver, crie um VG para praticar este exercício:
# sudo pvcreate /dev/sdb # Use um disco disponível, ex: sdb
# sudo vgcreate vg_system /dev/sdb

# 1. Crie um Volume Lógico (LV) de 1GB para swap chamado lv_swap no VG 'vg_system'.
sudo lvcreate -n lv_swap -L 1G vg_system

# 2. Formate lv_swap como área de swap.
sudo mkswap /dev/vg_system/lv_swap

# 3. Habilite o lv_swap.
sudo swapon /dev/vg_system/lv_swap

# Verificação:
sudo swapon --show
free -h

# 4. Configure o sistema para ativar lv_swap automaticamente no boot.
# Obtenha o UUID do swap LV:
UUID_SWAP=$(sudo blkid -s UUID -o value /dev/vg_system/lv_swap)
echo "UUID do swap LV: $UUID_SWAP"

# Adicione a entrada ao /etc/fstab:
# Abra o /etc/fstab com um editor de texto (vi/nano):
sudo vi /etc/fstab
# Adicione a seguinte linha:
# UUID=[COLE_SEU_UUID_SWAP_AQUI] swap swap defaults 0 0

# Teste o fstab para garantir que não há erros:
sudo mount -a

# 5. Desabilite a área de swap recém-criada.
sudo swapoff /dev/vg_system/lv_swap

# Verificação:
sudo swapon --show

# 6. Remova o lv_swap.
# Primeiro, remova a entrada do fstab para evitar problemas no boot.
# Abra o /etc/fstab e delete a linha do swap LV.
sudo vi /etc/fstab

# Em seguida, remova o LV:
sudo lvremove /dev/vg_system/lv_swap -y

# Verificação:
sudo lvs

# 7. Extra: Se você tem um LV de dados (ex: /dev/vg_data/lv_web_content),
# estenda-o em 500MB e redimensione o sistema de arquivos.
# Supondo que 'lv_web_content' está em 'vg_data' e montado em '/var/www/html/site'
# E o sistema de arquivos é XFS (se for ext4, os comandos de resize são diferentes)

# Estenda o Logical Volume em 500MB:
sudo lvextend -L +500M /dev/vg_data/lv_web_content

# Verificação do LV:
sudo lvs

# Redimensione o sistema de arquivos (para XFS):
sudo xfs_growfs /var/www/html/site

# Verificação do espaço em disco:
df -h /var/www/html/site

# Se o sistema de arquivos fosse Ext4:
# sudo resize2fs /dev/vg_data/lv_web_content
```

-----

## Exercício 07: Controle de Acesso com ACLs (Access Control Lists)

### Solução:

```bash
# Certifique-se de que os pacotes 'acl' estão instalados (geralmente vêm por padrão).
sudo dnf install -y acl

# 1. Crie um diretório /dados/compartilhado.
sudo mkdir -p /dados/compartilhado

# 2. Crie um usuário user_a e um usuário user_b.
sudo useradd user_a
echo "redhat" | sudo passwd --stdin user_a
sudo useradd user_b
echo "redhat" | sudo passwd --stdin user_b

# 3. Crie um arquivo /dados/compartilhado/documento.txt.
sudo touch /dados/compartilhado/documento.txt

# 4. Defina as permissões ACL para /dados/compartilhado.
# Remove todas as ACLs existentes primeiro (opcional, para garantir um estado limpo)
sudo setfacl -b /dados/compartilhado

# Definir ACLs para o diretório
# user_a: rwx
# user_b: r-x
# Outros: ---
sudo setfacl -m u:user_a:rwx /dados/compartilhado
sudo setfacl -m u:user_b:r-x /dados/compartilhado
sudo setfacl -m o::--- /dados/compartilhado # Define para outros sem acesso

# Verificação das ACLs (o '+' indica que há ACLs no diretório):
ls -ld /dados/compartilhado
getfacl /dados/compartilhado

# 5. Configure o diretório /dados/compartilhado para que as permissões ACLs sejam hereditárias.
# (d: significa default ACL)
sudo setfacl -m d:u:user_a:rwx /dados/compartilhado
sudo setfacl -m d:u:user_b:r-x /dados/compartilhado
sudo setfacl -m d:o::--- /dados/compartilhado

# Verificação:
getfacl /dados/compartilhado
# Você deve ver entradas de "default" no final.

# 6. Como user_a, crie um arquivo dentro de /dados/compartilhado e verifique suas permissões.
su - user_a <<'EOF'
echo "Conteúdo do user_a" > /dados/compartilhado/arquivo_user_a.txt
EOF
ls -l /dados/compartilhado/arquivo_user_a.txt
getfacl /dados/compartilhado/arquivo_user_a.txt
# O arquivo deve ter user_a com rwx e user_b com r-x (herdados do default ACL).
exit # Volte para o usuário root/sudo

# 7. Como user_b, tente criar um arquivo e editar documento.txt. Verifique o resultado.
su - user_b <<'EOF'
# Tentar criar um arquivo (deve falhar se as permissões de grupo forem as padrão ou o 'other' tiver '-')
echo "Tentando criar como user_b" > /dados/compartilhado/arquivo_user_b.txt
# Mensagem de erro esperada: Permission denied (Permissão negada)

# Tentar editar documento.txt (deve falhar, pois user_b só tem leitura e execução, não escrita)
echo "Tentando editar como user_b" >> /dados/compartilhado/documento.txt
# Mensagem de erro esperada: Permission denied (Permissão negada)

# Tentar ler documento.txt (deve funcionar)
cat /dados/compartilhado/documento.txt
EOF
exit # Volte para o usuário root/sudo

echo "Verifique os resultados dos comandos de user_b acima."
ls -l /dados/compartilhado/
getfacl /dados/compartilhado/documento.txt

# Limpeza (opcional, para resetar o ambiente)
# sudo setfacl -b /dados/compartilhado
# sudo userdel -r user_a
# sudo userdel -r user_b
# sudo rm -rf /dados/compartilhado
```

-----

## Exercício 08: Gerenciamento do Boot e Redefinição de Senha

### Solução:

Este exercício requer interação direta com a console da VM ou máquina física.

#### **Passo 1: Simular senha de root esquecida (Opcional)**

  * Faça login como `root` ou um usuário com `sudo` e altere a senha de `root` para algo que você "esqueça".
    ```bash
    sudo passwd root
    # Digite uma senha aleatória para simular "esquecimento"
    ```
  * Tente fazer login como `root` com a senha antiga para confirmar que não funciona.

#### **Passo 2: Reinicialize e acesse o menu do GRUB**

1.  Reinicialize o sistema:
    ```bash
    sudo reboot
    ```
2.  Durante o boot, quando o menu do GRUB aparecer, **pressione a tecla `e`** rapidamente na entrada de boot padrão do RHEL.

#### **Passo 3: Modifique a entrada do GRUB**

1.  Você estará em um editor de texto simples. Procure a linha que começa com `linux` ou `linuxefi`.
2.  Nesta linha, localize o parâmetro `ro` (read-only).
3.  Altere `ro` para `rw` (read-write).
4.  Adicione `init=/bin/bash` (ou `init=/bin/sh`) no final desta mesma linha.
      * **Exemplo:**
        ```
        linuxefi /vmlinuz-... ro crashkernel=auto ... rhgb quiet  # Original
        linuxefi /vmlinuz-... rw crashkernel=auto ... rhgb quiet init=/bin/bash # Modificado
        ```
5.  Pressione **`Ctrl-x`** ou **`F10`** para iniciar o sistema com as modificações.

#### **Passo 4: Monte o sistema de arquivos raiz (/) em modo de leitura/escrita**

1.  O sistema inicializará diretamente para um shell de `root` (`bash` ou `sh`).
2.  O sistema de arquivos raiz pode estar montado apenas como leitura. Confirme:
    ```bash
    mount | grep " / "
    # Se vir 'ro', precisaremos remontar.
    ```
3.  Remonte o sistema de arquivos raiz em modo de leitura/escrita:
    ```bash
    mount -o remount,rw /sysroot
    ```

#### **Passo 5: Redefina a senha de root**

1.  Agora que o sistema de arquivos raiz está montado com permissão de escrita, você pode redefinir a senha do `root`.
    ```bash
    chroot /sysroot
    passwd root
    # Digite "redhat" e confirme.
    ```
      * Você deve ver a mensagem "all authentication tokens updated successfully."

#### **Passo 6: Aplique o relabeling do SELinux**

1.  Se o SELinux estiver em modo `enforcing`, uma mudança de senha pode ter consequências de contexto de segurança. É uma boa prática forçar um relabeling completo na próxima inicialização para evitar problemas.
    ```bash
    touch /.autorelabel
    ```

#### **Passo 7: Reinicialize o sistema e faça login como root com a nova senha**

1.  Sincronize as mudanças e reinicialize o sistema:
    ```bash
    sync
    exec /sbin/init # Ou reboot -f
    ```
2.  O sistema irá reiniciar. Se você adicionou `.autorelabel`, ele passará por um processo de relabeling do SELinux (isso pode levar tempo).
3.  Após a reinicialização completa, tente fazer login como `root` usando a nova senha **`redhat`**.

-----

## Exercício 09: Gerenciamento de Arquivos Temporários com `systemd-tmpfiles`

### Solução:

```bash
# --- Configuração Inicial: Criar Usuário e Grupo ---
# 1. Crie o usuário e o grupo 'minhaapp'
sudo groupadd minhaapp
sudo useradd -g minhaapp minhaapp

# Verifique se foram criados
id minhaapp
grep minhaapp /etc/group

# --- Preparação para o Teste: Simular Diretórios e Arquivos ---
# Crie os diretórios que serão gerenciados
sudo mkdir -p /var/log/minhaapp
sudo mkdir -p /var/cache/minhaapp

# Mude a propriedade para teste (o tmpfiles vai sobrescrever depois)
sudo chown root:root /var/cache/minhaapp
sudo chmod 755 /var/cache/minhaapp

# Crie alguns arquivos de teste para simular logs e cache
sudo touch /var/log/minhaapp/log_antigo_1.log
sudo touch /var/log/minhaapp/log_antigo_2.log
sudo touch /var/log/minhaapp/log_recente.log
sudo touch /var/cache/minhaapp/cache_file.tmp

# Altere a data de modificação de alguns arquivos para simular "antigos"
# Isso é para testar a regra de 7 dias. Você pode ajustar o 'date' conforme necessário.
sudo touch -d "8 days ago" /var/log/minhaapp/log_antigo_1.log
sudo touch -d "10 days ago" /var/log/minhaapp/log_antigo_2.log

echo "Arquivos antes da limpeza:"
ls -l /var/log/minhaapp/
ls -l /var/cache/minhaapp/

# --- Configuração do systemd-tmpfiles ---
# 2. Crie um arquivo de configuração para systemd-tmpfiles
# Crie o arquivo em /etc/tmpfiles.d/ (preferencial para configurações personalizadas)
sudo vi /etc/tmpfiles.d/minhaapp.conf # Ou 'sudo nano /etc/tmpfiles.d/minhaapp.conf'

# Adicione o seguinte conteúdo ao arquivo:
# Type Path                       Mode UID      GID      Age      Argument
d /var/cache/minhaapp        0770 minhaapp minhaapp -        -
Z /var/cache/minhaapp        -    minhaapp minhaapp -        -
D /var/log/minhaapp/* -    -        -        7d       -

# Salve e feche o arquivo.

# --- Testando as Regras ---

# 4. Execute systemd-tmpfiles em modo de limpeza para verificar exclusão
# Este comando simula a limpeza. Use '--dry-run' para ver o que seria feito sem realmente executar.
echo -e "\n--- Simulando limpeza (dry-run): ---"
sudo systemd-tmpfiles --clean --dry-run /etc/tmpfiles.d/minhaapp.conf

# Execute a limpeza real
echo -e "\n--- Executando limpeza real: ---"
sudo systemd-tmpfiles --clean /etc/tmpfiles.d/minhaapp.conf

echo -e "\nArquivos em /var/log/minhaapp/ após a limpeza:"
ls -l /var/log/minhaapp/
# Você deve ver apenas 'log_recente.log' (ou o que não tiver mais de 7 dias)

# 5. Execute systemd-tmpfiles em modo de criação/verificação
# Este comando garante que os diretórios existam com as permissões corretas.
# Primeiro, remova o diretório de cache para testar a recriação.
sudo rm -rf /var/cache/minhaapp

echo -e "\n--- Executando criação/verificação: ---"
sudo systemd-tmpfiles --create /etc/tmpfiles.d/minhaapp.conf
# ou simplesmente:
# sudo systemd-tmpfiles --create

echo -e "\nDiretórios e permissões após a criação/verificação:"
ls -ld /var/cache/minhaapp/
# Saída esperada: drwxrwx---. 2 minhaapp minhaapp ... /var/cache/minhaapp/

# Verifique o conteúdo do diretório de cache (deve estar vazio, mas o diretório existe)
ls -l /var/cache/minhaapp/
```

-----

## Exercício 10: Gerando e Gerenciando um Serviço Podman com Systemd

### Solução:

```bash
# --- 1. Criar e Iniciar o Contêiner MariaDB ---

# Puxe a imagem MariaDB (se não tiver)
echo "Passo 1: Puxando a imagem mariadb:latest..."
sudo podman pull mariadb:latest

# Verifique as imagens
sudo podman images

# Crie e inicie o contêiner MariaDB
# Usamos 'mariadb:latest' e definimos uma senha de root para o banco de dados.
# O '-d' é para rodar em segundo plano (detached).
echo "Passo 1: Criando e iniciando o contêiner my-mariadb-container..."
sudo podman run -d --name my-mariadb-container -e MARIADB_ROOT_PASSWORD=mysecretpassword mariadb:latest

# Verifique se o contêiner está em execução
echo "Passo 1: Verificando o status do contêiner Podman..."
sudo podman ps

# --- 2. Gerar a Unidade Systemd ---

# Use podman generate systemd para criar o arquivo .service
# A opção '--files' cria o arquivo no diretório atual.
# A opção '--name' indica para gerar a unidade para um contêiner específico.
echo "Passo 2: Gerando o arquivo de unidade systemd para my-mariadb-container..."
podman generate systemd --name my-mariadb-container --files

# Saída esperada (o nome do arquivo gerado será o nome do contêiner com .service):
# /home/seu_usuario/container-my-mariadb-container.service

# Verifique o conteúdo do arquivo gerado
echo "Passo 2: Conteúdo do arquivo de unidade systemd gerado:"
cat container-my-mariadb-container.service

# Você verá um arquivo .service com seções como [Unit], [Service], [Install].
# Note que ele já inclui os comandos `podman start` e `podman stop`.

# --- 3. Configurar e Habilitar o Serviço Systemd ---

# Mova o arquivo da unidade para o diretório de serviços do systemd.
# Para serviços gerenciados pelo sistema (todos os usuários), o local é /etc/systemd/system/.
# Se fosse para um usuário específico, seria ~/.config/systemd/user/.
echo "Passo 3: Movendo o arquivo .service para /etc/systemd/system/..."
sudo mv container-my-mariadb-container.service /etc/systemd/system/

# Recarregue o daemon do systemd para que ele reconheça o novo serviço.
echo "Passo 3: Recarregando o daemon do systemd..."
sudo systemctl daemon-reload

# Habilite o serviço para iniciar automaticamente na inicialização do sistema.
echo "Passo 3: Habilitando o serviço para iniciar no boot..."
sudo systemctl enable container-my-mariadb-container.service

# Inicie o serviço MariaDB agora usando systemctl.
# Isso irá parar o contêiner gerenciado diretamente pelo podman e iniciá-lo via systemd.
echo "Passo 3: Iniciando o serviço my-mariadb-container via systemctl..."
sudo systemctl start container-my-mariadb-container.service

# --- 4. Verifique o Status do Serviço ---

# Verifique o status do serviço systemd.
echo "Passo 4: Verificando o status do serviço systemd..."
sudo systemctl status container-my-mariadb-container.service
# A saída deve mostrar 'Active: active (running)'.

# Verifique se o contêiner está em execução via podman (agora gerenciado pelo systemd).
echo "Passo 4: Verificando o status do contêiner Podman (deve estar em execução)..."
sudo podman ps
# Você ainda deve ver o contêiner 'my-mariadb-container' listado.

# --- 5. Testar Persistência e Limpeza (Opcional, mas recomendado) ---

# Teste de persistência: Pare o contêiner diretamente com Podman.
# O systemd, por padrão, tentará reiniciá-lo se a opção 'Restart=' estiver configurada na unidade.
echo "Passo 5: Testando persistência - Parando o contêiner via Podman (systemd deve reiniciar)..."
sudo podman stop my-mariadb-container

# Dê alguns segundos e verifique o status novamente.
sleep 5
echo "Passo 5: Verificando status após parada manual (deve ter reiniciado):"
sudo systemctl status container-my-mariadb-container.service
sudo podman ps
# Você deve notar que ele reiniciou automaticamente.

# Desabilite e pare o serviço systemd
echo "Passo 5: Desabilitando e parando o serviço systemd para limpeza..."
sudo systemctl disable container-my-mariadb-container.service
sudo systemctl stop container-my-mariadb-container.service

# Verifique se o serviço parou
sudo systemctl status container-my-mariadb-container.service
sudo podman ps # O contêiner também deve ter parado.

# Remova o contêiner
echo "Passo 5: Removendo o contêiner para limpeza..."
sudo podman rm my-mariadb-container

# Remova o arquivo da unidade systemd
echo "Passo 5: Removendo o arquivo da unidade systemd para limpeza..."
sudo rm /etc/systemd/system/container-my-mariadb-container.service

# Recarregue o daemon do systemd novamente para remover a unidade da memória
echo "Passo 5: Recarregando o daemon do systemd pela última vez..."
sudo systemctl daemon-reload

echo "Exercício concluído!"
```

## Solução Detalhada

Vamos passo a passo para configurar ambos os lados.

### Parte 1: Configuração do Servidor NFS (`192.168.xxx.10`)

```bash
# 1. Instale os pacotes necessários para o servidor NFS.
sudo dnf install -y nfs-utils

# 2. Crie um diretório para ser compartilhado via NFS.
sudo mkdir -p /srv/nfs_share

# 3. Defina permissões e propriedade adequadas para o diretório compartilhado.
# Geralmente, permissão 777 é usada para testes ou colaboração ampla.
# Para um ambiente mais restritivo, você definiria um grupo específico.
sudo chmod 777 /srv/nfs_share
sudo chown nobody:nobody /srv/nfs_share # Define 'nobody' para evitar problemas de ID de usuário

# 4. Configure o FirewallD para permitir o tráfego NFS.
# Adicione os serviços NFS, mountd e rpc-bind permanentemente.
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd

# Recarregue o firewall para aplicar as regras.
sudo firewall-cmd --reload

# Verifique se os serviços NFS estão abertos no firewall.
sudo firewall-cmd --list-all | grep services

# 5. Exporte o diretório /srv/nfs_share para ser acessível pelo cliente NFS.
# Edite o arquivo /etc/exports.
sudo vi /etc/exports # ou sudo nano /etc/exports

# Adicione a seguinte linha no final do arquivo:
# Substitua 192.168.xx.20 pelo IP real do seu cliente NFS.
# /srv/nfs_share 192.168.xx.20(rw,sync,no_root_squash)
# Explicação das opções:
#   rw: leitura e escrita
#   sync: garante que as alterações sejam gravadas no disco antes de responder ao cliente
#   no_root_squash: permite que o usuário root do cliente acesse o compartilhamento como root no servidor (use com cautela)

# Exemplo completo da linha para /etc/exports:
# /srv/nfs_share 192.168.xx.20(rw,sync,no_root_squash)

# 6. Reinicie e habilite o serviço NFS no boot.
# O nfs-server.service lida com os serviços relacionados ao NFS.
sudo systemctl enable --now nfs-server

# Verifique o status do serviço.
sudo systemctl status nfs-server

# 7. Verifique as exportações do servidor.
# Isso mostra os compartilhamentos configurados e para quem.
exportfs -v
```

-----

### Parte 2: Configuração do Cliente NFS (`192.168.xx.20`)

```bash
# 1. Instale os pacotes necessários para o cliente NFS.
sudo dnf install -y nfs-utils

# 2. Crie um ponto de montagem para o compartilhamento NFS.
sudo mkdir -p /mnt/nfs_docs

# 3. Monte o compartilhamento NFS do servidor no ponto de montagem do cliente.
# Substitua 192.168.xxx.10 pelo IP real do seu servidor NFS.
sudo mount 192.168.xxx.10:/srv/nfs_share /mnt/nfs_docs

# Verifique se a montagem foi bem-sucedida.
df -h /mnt/nfs_docs
# Você deve ver o compartilhamento montado.

# 4. Configure a montagem automática do compartilhamento NFS na inicialização do sistema usando /etc/fstab.
# Edite o arquivo /etc/fstab.
sudo vi /etc/fstab # ou sudo nano /etc/fstab

# Adicione a seguinte linha no final do arquivo:
# Substitua 192.168.xxx.10 pelo IP real do seu servidor NFS.
# 192.168.xxx.10:/srv/nfs_share /mnt/nfs_docs nfs defaults,_netdev 0 0
# Explicação das opções:
#   nfs: tipo de sistema de arquivos
#   defaults: opções padrão (rw, suid, dev, exec, auto, nouser, async)
#   _netdev: garante que a montagem ocorra apenas após a rede estar disponível

# Exemplo completo da linha para /etc/fstab:
# 192.168.xxx.10:/srv/nfs_share /mnt/nfs_docs nfs defaults,_netdev 0 0

# Teste a entrada do fstab sem reiniciar.
# Primeiro, desmonte o compartilhamento para garantir que o 'mount -a' irá montá-lo.
sudo umount /mnt/nfs_docs
sudo mount -a
# Se não houver erros, nenhum output será exibido.

# 5. Teste a conectividade e a capacidade de leitura/escrita no compartilhamento montado.
# Crie um arquivo no compartilhamento como um usuário comum (se tiver permissão).
# Se você usou 'no_root_squash' no servidor, o root do cliente pode escrever.
sudo touch /mnt/nfs_docs/teste_do_cliente.txt
echo "Isso é um teste de escrita do cliente." | sudo tee /mnt/nfs_docs/teste_escrita.txt

# Verifique se os arquivos foram criados no cliente.
ls -l /mnt/nfs_docs/

# Verifique no servidor se os arquivos do cliente apareceram (opcional, mas recomendado).
# No servidor (192.168.xxx.10):
# ls -l /srv/nfs_share/

# Limpeza (opcional):
# No cliente:
# sudo umount /mnt/nfs_docs
# (Remova a linha do /etc/fstab)
# sudo rmdir /mnt/nfs_docs

# No servidor:
# (Remova a linha do /etc/exports)
# sudo exportfs -ra # Recarregue as exportações
# sudo rmdir /srv/nfs_share
# sudo dnf remove -y nfs-utils
```
