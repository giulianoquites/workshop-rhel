## Exercício 1: Gerenciamento de Usuários, Grupos e Permissões de Colaboração

A equipe de desenvolvimento precisa de uma nova estrutura de acesso para um projeto.

### Objetivos:

* Crie um grupo chamado **`projetodelta`**.
* Crie dois usuários: **`devuser1`** e **`devuser2`**. Ambos devem ter a senha **`redhat123`**.
* Adicione **`devuser1`** e **`devuser2`** ao grupo **`projetodelta`** como grupo secundário.
* Crie um diretório **/srv/projetodelta**.
* Defina o proprietário do diretório **/srv/projetodelta** para **`root`** e o grupo para **`projetodelta`**.
* Defina as permissões para **/srv/projetodelta** de modo que:
    * O proprietário (**`root`**) tenha permissões totais (**`rwx`**).
    * Membros do grupo **`projetodelta`** tenham permissões de leitura e escrita (**`rw-`**) e possam entrar no diretório (**`x`**).
    * Outros usuários não tenham nenhuma permissão.
* Configure o diretório **/srv/projetodelta** para que todos os arquivos e subdiretórios criados dentro dele herdem o grupo **`projetodelta`** automaticamente.
* Como **`devuser1`**, crie um arquivo **/srv/projetodelta/relatorio_dev1.txt** e adicione algum texto.
* Verifique se o arquivo **`relatorio_dev1.txt`** tem o grupo **`projetodelta`**.

---

## Exercício 2: Gerenciamento de Armazenamento Local (LVM)

### Cenário:
Você precisa expandir o armazenamento disponível para um novo serviço e gerenciá-lo com LVM.

### Pré-requisito:
Adicione um novo disco virtual à sua VM (ex: **/dev/sdb**) de pelo menos 5GB.

### Objetivos:

* Crie um Volume Físico (PV) no novo disco **/dev/sdb**.
* Crie um Grupo de Volumes (VG) chamado **`vg_data`** usando o PV recém-criado.
* Crie um Volume Lógico (LV) chamado **`lv_web_content`** de 2GB dentro de **`vg_data`**.
* Formate **`lv_web_content`** com o sistema de arquivos **XFS**.
* Monte **`lv_web_content`** em **/var/www/html/site**.
* Configure o sistema para montar **`lv_web_content`** automaticamente no boot, usando o UUID do sistema de arquivos.
* Estenda **`lv_web_content`** em 1GB.
* Estenda o sistema de arquivos XFS para usar o novo espaço.

---

## Exercício 3: Configuração de Rede e FirewallD

### Cenário:
Você precisa configurar o sistema com um IP estático e garantir que os serviços de rede estejam protegidos.

### Objetivos:

* Configure a interface de rede principal (ex: `ens192`, `eth0` - use `ip a` para verificar o nome) com as seguintes configurações estáticas:
    * Endereço IP: **`192.168.100.10/24`**
    * Gateway: **`192.168.100.1`**
    * Servidor DNS: **`8.8.8.8`**
* Defina o hostname do sistema para **`servidorweb.example.com`**.
* Reinicie o serviço de rede para aplicar as mudanças.
* Verifique se o hostname e as configurações de IP foram aplicados.
* Abra a porta **`80`** (HTTP) e **`443`** (HTTPS) no FirewallD permanentemente.
* Recarregue o FirewallD para aplicar as regras imediatamente.
* Verifique se as portas estão abertas.
* Instale o pacote **`httpd`** (Apache) e configure-o para iniciar automaticamente no boot.
* Crie um arquivo **`index.html`** simples em **/var/www/html** e verifique o acesso via **`curl localhost`**.

---

## Exercício 4: Agendamento de Tarefas (Cron) e Monitoramento de Logs

### Cenário:
Você precisa automatizar uma tarefa diária e monitorar os logs do sistema.

### Objetivos:

* Crie um script simples **`monitora_memoria.sh`** em **/usr/local/bin** que:
    * Obtenha a quantidade de memória livre (**`free -h`**).
    * Grave essa informação em um arquivo de log **/var/log/memoria.log**, adicionando a data e hora de cada entrada.
* Configure uma tarefa **Cron** para que o script **`monitora_memoria.sh`** seja executado todos os dias à **01:00 AM**.
* Verifique se a tarefa Cron foi agendada corretamente para o usuário **`root`**.
* Configure a rotação de logs para **/var/log/memoria.log** usando **`logrotate`**, para que ele seja rotacionado semanalmente, mantendo 4 arquivos de rotação e comprimindo os antigos.
* Use **`journalctl`** para verificar os logs do serviço **`crond`** para confirmar a execução da tarefa.

---

## Exercício 5: Gerenciamento de Serviços (Systemd) e Processos

### Cenário:
Você precisa gerenciar e monitorar um serviço web, garantindo sua inicialização e resolvendo problemas.

### Objetivos:

* Instale o pacote **`nginx`**.
* Habilite o serviço **`nginx`** para iniciar no boot.
* Inicie o serviço **`nginx`**.
* Verifique o status do serviço **`nginx`** e garanta que ele está ativo e rodando.
* Pare o serviço **`nginx`**.
* Reinicie o serviço **`nginx`**.
* Identifique o processo principal do **`nginx`** (o processo pai) e veja seu PID.
* Altere a prioridade (**`nice value`**) do processo principal do **`nginx`** para **`-5`** (mais prioritário).
* Verifique a nova prioridade do processo.
* Desabilite o serviço **`nginx`** para que ele não inicie automaticamente no boot.

---

## Exercício 6: Gerenciamento de Armazenamento Local - Swap e Redimensionamento

### Cenário:
Você precisa gerenciar o espaço de swap e expandir um volume lógico existente.

### Objetivos:

* Crie um Volume Lógico (LV) de 1GB para swap chamado **`lv_swap`** em um Volume Group existente (ou crie um VG se não tiver, ex: `vg_system` usando `/dev/vdb` se disponível).
* Formate **`lv_swap`** como área de swap.
* Habilite o **`lv_swap`**.
* Configure o sistema para ativar **`lv_swap`** automaticamente no boot.
* Desabilite a área de swap recém-criada.
* Remova o **`lv_swap`**.
* **Extra:** Se você tem um LV de dados (do exercício anterior, ex: `/dev/vg_data/lv_web_content`), estenda-o em 500MB e redimensione o sistema de arquivos (XFS ou Ext4) para usar o novo espaço.

---

## Exercício 7: Controle de Acesso com ACLs (Access Control Lists)

### Cenário:
Você precisa fornecer permissões granulares para usuários e grupos em um diretório, além das permissões padrão do Linux.

### Objetivos:

* Crie um diretório **/dados/compartilhado**.
* Crie um usuário **`user_a`** e um usuário **`user_b`**.
* Crie um arquivo **/dados/compartilhado/documento.txt**.
* Defina as permissões ACL para **/dados/compartilhado** de modo que:
    * **`user_a`** tenha permissão total (**`rwx`**).
    * **`user_b`** tenha permissão de leitura e execução (**`r-x`**).
    * Outros usuários (que não `user_a` nem `user_b`) não tenham acesso.
* Configure o diretório **/dados/compartilhado** para que as permissões ACLs sejam hereditárias para novos arquivos e diretórios criados dentro dele.
* Como **`user_a`**, crie um arquivo dentro de **/dados/compartilhado** e verifique suas permissões.
* Como **`user_b`**, tente criar um arquivo e editar **`documento.txt`**. Verifique o resultado.

---

## Exercício 8: Gerenciamento do Boot e Redefinição de Senha

### Cenário:
O sistema não está inicializando corretamente e você precisa redefinir a senha de root.

### Objetivos:

* Simule um cenário de senha de root esquecida.
    * (Opcional, mas útil para praticar) Altere a senha de root para algo que você "esqueça".
* Reinicialize a máquina e acesse o menu do GRUB (geralmente pressionando a tecla `e` na entrada de boot do RHEL).
* Modifique a entrada do GRUB para inicializar em modo de usuário único/recuperação.
* Monte o sistema de arquivos raiz (/) em modo de leitura/escrita.
* Redefina a senha de root para **`redhat`**.
* Aplique o relabeling do SELinux (**`autorelabel`**) para garantir que as mudanças de contexto de segurança sejam aplicadas após a reinicialização.
* Reinicialize o sistema e faça login como root com a nova senha.

---

## Exercício 9: Gerenciamento de Arquivos Temporários com `systemd-tmpfiles`

### Cenário:
Você é um administrador de sistemas e precisa garantir que o diretório de log de uma aplicação específica (`/var/log/minhaapp`) seja limpo regularmente. Além disso, a aplicação precisa que um diretório de cache (`/var/cache/minhaapp`) exista com permissões específicas e seja propriedade de um usuário e grupo dedicados, sendo recriado automaticamente se for excluído.

### Objetivos:

* Crie o usuário e o grupo **`minhaapp`**.
* Crie um arquivo de configuração para **`systemd-tmpfiles`** que:
    * Garanta que o diretório **/var/cache/minhaapp** exista, seja propriedade do usuário **`minhaapp`** e do grupo **`minhaapp`**, com permissões **`0770`**.
    * Garanta que todos os arquivos e subdiretórios dentro de **/var/log/minhaapp** sejam limpos (excluídos) automaticamente após 7 dias da última modificação.
* Simule alguns arquivos de log e cache para testar as regras.
* Execute **`systemd-tmpfiles`** em modo de limpeza para verificar se as regras de exclusão funcionam.
* Execute **`systemd-tmpfiles`** em modo de criação/verificação para garantir que os diretórios sejam criados ou tenham as permissões corretas.

---

## Exercício 10: Gerando e Gerenciando um Serviço Podman com Systemd

### Cenário:
Você tem um contêiner MariaDB em execução no seu sistema RHEL, gerenciado pelo Podman. Para garantir que este serviço inicie automaticamente na inicialização do sistema e seja gerenciável via `systemctl`, você precisa gerar uma unidade de serviço `systemd` para ele.

### Pré-requisitos:

* Um sistema RHEL com **Podman** instalado.
* Uma imagem **MariaDB** disponível (será baixada se não estiver presente).
* Um contêiner MariaDB em execução (ou que possa ser iniciado).

### Tarefas:

* **Criar e Iniciar o Contêiner MariaDB:**
    * Puxe a imagem **`mariadb:latest`** (se ainda não tiver).
    * Crie e inicie um contêiner MariaDB chamado **`my-mariadb-container`** com uma variável de ambiente para a senha do root.
* **Gerar a Unidade Systemd:**
    * Use **`podman generate systemd`** para criar um arquivo de unidade `systemd` para o contêiner **`my-mariadb-container`**.
    * Verifique o conteúdo do arquivo gerado.
* **Configurar e Habilitar o Serviço Systemd:**
    * Mova o arquivo da unidade `systemd` para o local apropriado.
    * Recarregue a configuração do `systemd`.
    * Habilite o serviço para iniciar no boot.
    * Inicie o serviço usando **`systemctl`**.
* **Verificar o Status do Serviço:**
    * Confirme que o serviço **`my-mariadb-container.service`** está ativo e em execução via `systemctl`.
    * Verifique se o contêiner está em execução via **`podman`**.
* **Testar Persistência (Opcional, mas recomendado):**
    * Pare o contêiner diretamente com **`podman stop`**.
    * Verifique se o `systemd` o reinicia automaticamente.
    * Desabilite e pare o serviço `systemd`.
    * Remova o contêiner e o arquivo da unidade `systemd` para limpeza.

---

## Exercício 11: Configuração de Servidor e Cliente NFS

### Cenário:

Você tem dois servidores RHEL. Um deles será configurado como um **servidor NFS** para compartilhar um diretório de documentos, e o outro será configurado como um **cliente NFS** para montar esse diretório compartilhado e acessá-lo.

### Pré-requisitos:

  * Duas máquinas RHEL (virtuais ou físicas), uma para o servidor e outra para o cliente.
  * Conectividade de rede entre elas. Anote os endereços IP de cada uma.
      * **Servidor NFS:** Ex: `192.168.100.10`
      * **Cliente NFS:** Ex: `192.168.100.20`
  * Acesso `root` ou `sudo` em ambas as máquinas.

### Objetivos:

**No Servidor NFS (`192.168.100.10`):**

1.  Instale os pacotes necessários para o servidor NFS.
2.  Crie um diretório para ser compartilhado via NFS (ex: `/srv/nfs_share`).
3.  Defina permissões e propriedade adequadas para o diretório compartilhado.
4.  Configure o FirewallD para permitir o tráfego NFS.
5.  Exporte o diretório `/srv/nfs_share` para ser acessível pelo cliente NFS (`192.168.100.20`), permitindo leitura e escrita, e `sync` para garantir a gravação imediata.
6.  Reinicie e habilite o serviço NFS no boot.
7.  Verifique as exportações do servidor.

**No Cliente NFS (`192.168.100.20`):**

1.  Instale os pacotes necessários para o cliente NFS.
2.  Crie um ponto de montagem para o compartilhamento NFS (ex: `/mnt/nfs_docs`).
3.  Monte o compartilhamento NFS do servidor (`192.168.100.10:/srv/nfs_share`) no ponto de montagem do cliente.
4.  Configure a montagem automática do compartilhamento NFS na inicialização do sistema usando `/etc/fstab`.
5.  Teste a conectividade e a capacidade de leitura/escrita no compartilhamento montado.

-----