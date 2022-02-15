# Configuração da Infraestrutura

- [Lousa virtual](http://dontfile.com/curso513)
- [Repo do curso](https://github.com/4linux/513)

- [CLI do Vagrant](https://www.vagrantup.com/docs/cli/)

```console
vagrant up ansible-server
vagrant halt ansible-server
vagrant up
vagrant halt
vagrant destroy
vagrant destroy dbserver
vagrant up dbserver

vagrant ssh ansible-server # Preciso estar posicionado no dir em que está presente o Vagrantfile
```

# Conhecendo o Ansible

- Ferramentas de gerência de configuração (FGC)
  - Puppet e Chef são outras ferramentas
  - Provisionar: Ansible
  - Manter: Puppet
- Vem para substituir o método manual de configuração de ambientes
- Infraestrutura como código

  - Através das FGCs

- O Ansible é um framework open source que oferece recursos e ferramentas para implantar um modelo único de gerência de configurações em seu ambiente
- Quais são os ganhos com o Ansible?
  - Documentação instantânea;
  - Restore de backups e mudanças;
  - Processos bem definidos;
  - Ambiente padronizado;
  - Sistemas automatizados.

## A arquitetura do Ansible

<pre>
 Inventory                                    Componentes
    |                                         |__________ Server001
    V                                         |__________ Server002
 Playbook -> Ansible Config -> Python - SSH ->|__________ Server003
    ^                                         |__________ Server004
    |                                         |__________ Server005
 Modules                                      |__________ Server006
</pre>

- Vantagem: comunicação do Ansible com as outras máquinas é realizada pelo SSH
  - Não é necessário um agente instalado nos servidores alvos

## Instalando o ansible no Linux

```console
# Acessar a máquina ansible-server
$ ssh suporte@172.16.0.200
Passwd: 4linux

$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible -y
$ ansible --version

# Acessar o diretório do ansible
$ cd /etc/ansible
$ ls
ansible.cfg  hosts  roles
```

## Instalando o ansible no Windows

## Configurando o ansible (ansible.cfg)

```console
# Após uma determinada versão o ansible, é necessário gerar um arquivo modelo
$ sudo su
$ ansible-config init --disabled > ansible.cfg

# Editar
# Configurar as seguintes diretivas:
#  log_path=/var/log/ansible.log
#  private_key_file=/etc/keys/sshkeys
#  remote_user=root
#  roles_path=/etc/ansible/roles
#  timeout=30
$ vim ansible.cfg

```

## O SSH e chaves SSH no Ansible

```console
$ mkdir /etc/keys
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /etc/keys/sshkey
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /etc/keys/sshkey
Your public key has been saved in /etc/keys/sshkey.pub
The key fingerprint is:
SHA256:vEog7mA7iPWNKqeSJlzo4dMwavl0ICTDLNqS250dmto root@ansible-server
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|o                |
|++               |
|=+     .         |
|+.+.. . S        |
| Xo+.* . .       |
|X+Xo=+o .        |
|X%+++...         |
|*=BoE .          |
+----[SHA256]-----+

# Validar se a chave foi gerada
$ ls /etc/keys/
sshkey  sshkey.pub

# Enviar a chave pública para as máquinas alvo
$ ssh-copy-id -i /etc/keys/sshkey.pub ansible-server
$ ssh-copy-id -i /etc/keys/sshkey.pub webserver1
$ ssh-copy-id -i /etc/keys/sshkey.pub webserver2
$ ssh-copy-id -i /etc/keys/sshkey.pub dbserver

# Executar um comando via ssh sem precisar passar senha, passando a chave ssh
$ ssh -i /etc/keys/sshkey ansible-server cal
$ ssh -i /etc/keys/sshkey webserver1 cal
$ ssh -i /etc/keys/sshkey webserver2 cal
$ ssh -i /etc/keys/sshkey dbserver cal

```

## Configurar Inventários

```console
# Editar o invenvário
# Adicionar o conteúdo:
#   [local]
#   ansible-server
#   [webservers]
#   webserver[1:2]
#
#   [db]
#   dbserver
$ vim hosts

# Testar configuração
$ ansible local -m ping
ansible-server | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

$ ansible webservers -m ping
webserver2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
webserver1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

$ ansible db -m ping
dbserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

- Para nos comunicarmos com máquinas Windows:

```console
$ apt install pip -y
$ pip3 install pywinrm
Requirement already satisfied: pywinrm in /usr/lib/python3/dist-packages (0.3.0)

# Adicionar o seguinte conteúdo ao inventário:
#    [windows]
#    winclient
#    [windows:vars]
#    ansible_user=vagrant
#    ansible_password=vagrant
#    ansible_connection=winrm
#    ansible_port=5986
#    ansible_winrm_server_cert_validation=ignore
#    ansible_winrm_transport=ssl
$ vim hosts

# Acessar a máquina windows e no powershell executar:
$ ansible windows -m win_ping
winclient | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Ansible ad hoc

- Gerenciamento do Ansible via linha de comando
- Comandos standalone
- Sintaxe para o comando:

```console
ansible [hosts] -m <module> -a <parameters> --extra-vars <key>=<value>
```

### Ad hoc no Linux

#### Módulo user

```console
# Criar um usuário
$ ansible local -m user -a 'name=helpdesk state=present shell=/bin/bash password=$1$LzyjW.nw$Fl6eKZbFCbsLJ9KaNbfzu.'

# Validar se o usuário foi criado
$ getent passwd helpdesk

# Remover um usuário
ansible local -m user -a 'name=helpdesk state=absent remove=yes'
```

#### Módulo package

```console
# Instale pacotes usando os seguintes comandos
$ ansible local -m package -a 'name=htop state=present'
$ ansible local -m package -a 'name=elinks state=present'

# Verifique se os pacotes foram instalados
$ dpkg -l | egrep 'htop|elinks'

# Remover vários pacotes
$ ansible local -m package -a 'name=htop,elinks state=absent'

# Verifique se os pacotes foram removidos
$ dpkg -l | egrep 'htop|elinks'
```

#### Módulo file

```console
# Criar um arquivo
$ ansible local -m file -a 'path=/etc/nologin owner=root group=root mode=0644 state=touch'

# Remover o arquivo criado
$ ansible local -m file -a 'path=/etc/nologin state=absent'
```

### Ad hoc no Windows

#### Módulo ansible.windows.win_user

```console
# Crie o usuário através do módulo ansible.windows.win_user
$ ansible windows -m ansible.windows.win_user -a 'name=suporte state=present groups=Users password=4LinuxCursoAnsible'
```

#### Módulo win_chocolatey

Gerenciar pacotes utilizando o [Chocolatey](https://chocolatey.org/).

```console
# Instalar o pacote putty
$ ansible windows -m win_chocolatey -a 'name=putty state=present'
```

#### Módulo ansible.windows.win_file

Permite criar arquivos e pastas em SO Windows.

```console
# Criar uma pasta chamada "pacotes" no caminho "C:"
$ ansible windows -m ansible.windows.win_file -a 'path=C:\pacotes state=directory'
```

# Gerenciando Playbooks no Ansible

## Playbooks

O Ansible pode ser usado para executar scripts de configuração em grandes quantidades de máquinas ou para automatizar tarefas. Para isso, existem Playbooks - arquivos especializados que cabem muito bem no padrão de documentação do Ansible.

Os Playbooks do Ansible são um conjunto de comandos que podem ser executados por um ou mais controladores. Cada Playbook representa um conjunto de ações que devem ser executadas em um grupo de hosts.

Os Playbooks do Ansible são um conjunto de comandos que podem ser executados por um ou mais controladores. Cada Playbook representa um conjunto de ações que devem ser executadas em um grupo de hosts.

O Ansible só executa ações sobre hosts que tenham a configuração da condição necessária para o desempenho da tarefa.

### Componentes de um Playbook

- **hosts**
  - Máquina em que o Playbook será aplicado, a partir do inventário
- **Task**
  - Coleção de comandos que serão executados em uma ou mais máquinas
- **Handlers**
  - Mesma função de uma Task dentro de um Playbook
  - Reaproveitamento de tarefas
  - Um Handle será executado quando chamado por outra tarefa
  - Útil para ações secundárias, como iniciar um serviço após a instalação ou recarregar um serviço depois de uma alteração de configuração

```yaml
---
- name: Verify apache installation
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
    - name: Ensure apache is at the latest version
      ansible.builtin.yum:
        name: httpd
        state: latest

    - name: Write the apache config file
      ansible.builtin.template:
        src: /srv/httpd.j2
        dest: /etc/httpd.conf
      notify:
        - Restart apache

    - name: Ensure apache is running
      ansible.builtin.service:
        name: httpd
        state: started

  handlers:
    - name: Restart apache
      ansible.builtin.service:
        name: httpd
        state: restarted
```

- **Módulos**
  - Através dos módulos o Ansible usa fatos do sistema para determinar quais ações devem ser feitas para realizar uma tarefa
- **Fatos**
  - Informações do sistema
- **Variáveis**
  - Permite mais flexibilidade para Playbooks e Roles

### Gerenciar Playbooks no Ansible

Verificar a sintaxe de um Playbook:

```console
sudo ansible-playbook --syntax-check 1-adduser.yaml
```

Aplicar as configurações:

```console
sudo ansible-playbook 1-adduser.yaml
```

## Variáveis

Podem ser definidas e usadas em qualquer parte das linhas de comando, indepente do contexto em que estão sendo executadas.

**Tipos de variáveis**

- **vars**: declara um ou mais valores em uma variável através da diretiva _vars_;
- **vars_files**: declara um ou mais valores em um arquivo de variável através da diretiva _vars_files_;
- **register**: declara uma variável a partir de uma tarefa concluída;
- **--extra-vars**: declara um ou mais valores em uma variável através da flag _--extra-vars_ na linha de comando;
- **fatos**: utiliza informações do sistema chamadas de fatos como valores de variáveis

### Gerenciar Playbooks no Ansible através de variáveis

**Diretiva vars**

```yaml
---
- hosts: local
  vars:
    - usuario: linus
  tasks:
    - name: Adicionar usuario
      user: name={{ usuario }} state=present shell=/bin/bash password=$1$i5CwO/2J$JIaH55NqG10CDpYLqLAZf/
```

Para utilizar a variável como argumento de um módulo, declare entre chaves duplas {{ variável }}.

**Diretiva vars_files**

```yaml
---
# playbook
- hosts: local
  vars_files:
    - /home/suporte/playbooks/vars.yml
  tasks:
    - name: Instala pacotes atraves de variaveis
      apt: name={{ pacotes }} state=present update_cache=true

# arquivo vars.yml
pacotes:
  - elinks
  - wget
```

Para declarar uma variável a partir de um arquivo externo, adicione a opção vars_files, apontando a localização e nome do arquivo.

**Diretiva register**

```yaml
---
- hosts: local
  tasks:
    - name: Instala pacote NTP
      apt: name=ntp state=present update_cache=true
      register: ntp_installed
    - name: Define o arquivo de configuracao do servidor NTP
      when: ntp_installed is succeeded
      copy: src=/home/suporte/playbooks/ntp.conf dest=/etc/ntp.conf owner=root group=root mode=0644
```

Para declarar uma variável a partir de uma task, adicione a variável **register** com o valor da tarefa concluída. Essa mesma variável pode ser usada em outras tasks como condição de execução através da opção _when_.

**Flag --extra-vars**

```yaml
# 6-create-dir.yaml
---
- hosts: local
  tasks:
    - name: Adicionar estrutura de diretórios atraves de variavel
      file: dest={{ diretorios }} state=directory recurse=yes owner=root group=root mode=775
# recurse=yes -> permite criar as pastas de forma recursiva. Ou seja, no exemplo abaixo, serão criados os diretório www, html, intranet, caso não existam.
```

```console
$ sudo ansible-playbook --extra-vars "diretorios=/tmp/var/www/html/intranet" 6-create-dir.yaml
```

**Fatos**

```yaml
---
- hosts: webservers
  tasks:
    - name: Instala Apache no Debian/Ubuntu
      when: ansible_os_family == "Debian"
      apt: name=apache2 state=present update_cache=true
      register: apache2_installed
    - name: Instala Apache no CentOS
      when: ansible_os_family == "RedHat"
      yum: name=httpd state=present
      register: httpd_installed
```

A variável utilizada na condição _when_, _ansible_os_family_, é um fato.

Outra opção para buscar os fatos, é utilizar o módulo setup:

```console
$ sudo ansible db -m setup
```

{red}Troubleshooting do problema no repo com relação ao apache2 no webserver2:{red}

```console
$ sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
$ sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
$ dnf distro-sync
```

## Módulos (Ansible Module Index)

[Ansible Module Index](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html) é o índice oficial e módulos do Ansible. Ele é gerado a partir da documentação dos módulos do Ansible em docs.ansible.com, que por sua vez é gerado a partir do código-fonte no repo do Ansible.

### Gerenciando Módulos no Ansible

#### Módulos ansible.builtin.apt_key/ansible.builtin.apt_repository/apt

O objetivo desses três módulos é:

- Adicionar uma chave GPG para o repositório oficial do Docker;
- Adicionar repositório do Docker para distribuição Ubuntu;
- Instalar o Docker.

```yaml
---
- hosts: local
  tasks:
    - name: Adiciona chave GPG para o repositório oficial do Docker
      ansible.builtin.apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present
      register: apt_key_add
    - name: Adiciona repositório do Docker
      when: apt_key_add is succeeded
      ansible.builtin.apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
        state: present
    - name: Instala o Docker
      apt:
        name: docker-ce
        state: present
        update_cache: true
```

#### Módulos apt/copy/service

O objetivo desses três módulos é:

- Instalar o pacote Nginx;
- Copiar um arquivo de configuração do Nginx;
- Reiniciar o serviço do Nginx.

```yaml
---
- hosts: local
  tasks:
    - name: Instala servidor Nginx
      apt:
        name: nginx
        state: present
        update_cache: true
      register: nginx_installed
      notify:
        - Start Nginx
    - name: Define o arquivo de configuracao do servidor Nginx
      when: nginx_installed is succeeded
      copy: src=/home/suporte/modulos/files/nginx/nginx.conf dest=/etc/nginx/nginx.conf owner=root group=root mode=0644
      notify:
        - Restart Nginx
  handlers:
    - name: Start Nginx
      service: name=nginx state=started
    - name: Restart Nginx
      service: name=nginx state=restarted
```

## Handlers

Executará uma determinada ação quando uma task termina com um estado de _changed_.

No exemplo acima temos:

- Caso seja concluída a tarefa de instalar o pacote nginx, notifique (_notify_) o Handler _Start Nginx_;
- Caso seja concluída a tarefa de configurar o arquivo nginx.conf, notifique (_notify_) o Handler _Restart Nginx_.

## import_tasks

Permite incluir uma coleção de playbooks em um único arquivo.

Exemplo:

```yaml
---
# 3-apache.yaml
- hosts: webservers
  tasks:
    - name: Inclui task de instalação do Apache
      include_tasks: 3.1-install-apache.yaml
    - name: Inclui task de configuração do Apache
      include_tasks: 3.2-configure-apache.yaml

  handlers:
    - name: Restart Apache
      when: ansible_os_family == "Debian"
      ansible.builtin.service:
        name: apache2
        state: restarted

    - name: Restart Httpd
      when: ansible_os_family == "RedHat"
      ansible.builtin.service:
        name: httpd
        state: restarted
        enabled: yes

# 3.1-install-apache.yaml
- name: Instala pacotes para suporte a PHP no Apache em Distribuições Debian/Ubuntu
  when: ansible_os_family == "Debian"
  apt:
    state: present
    update_cache: true
    pkg:
      - apache2
      - php7.4
      - php7.4-mysql
      - php-memcache
  register: php_packages_ubuntu_installed
  notify:
    - Restart Apache

- name: Instala pacotes para suporte a PHP no Apache em Distribuições RedHat/CentOS
  when: ansible_os_family == "RedHat"
  yum:
    state: present
    pkg:
      - epel-release
      - httpd
      - php
      - php-mysqlnd
  register: php_packages_centos_installed
  notify:
    - Restart Httpd

# 3.2-configure-apache.yaml
- name: Remove o arquivo index.html
  when:
    - ansible_os_family == "Debian"
    - php_packages_ubuntu_installed is succeeded
  file:
    path: /var/www/html/index.html
    state: absent

- name: Define o arquivo de configuração para testar suporte PHP em Distribuições Debian/Ubuntu
  when:
    - ansible_os_family == "Debian"
    - php_packages_ubuntu_installed is succeeded
  copy: src=/home/suporte/modulos/files/web/index.php dest=/var/www/html/index.php owner=www-data group=www-data mode=0644

- name: Define o arquivo de configuração para testar o suporte PHP em Distribuições RedHat/CentOS
  when:
    - ansible_os_family == "RedHat"
    - php_packages_centos_installed is succeeded
  copy: src=/home/suporte/modulos/files/web/index.php dest=/var/www/html/index.php owner=apache group=apache mode=0644
```

# Gerenciando Roles no Ansible

[Ansible Galaxy](https://galaxy.ansible.com/)

## Gerenciar Roles

## Tipos de Roles

## Variáveis

## Criar Role com variáveis

# Gerenciando Templates no Ansible

## Introdução a Templates

## Utilizar Roles com templates

# Gerenciando Testes com Kitchen

## Conhecendo o Kitchen

## Instalar o Kitchen

## Gerenciar Roles com Kitchen

## Gerenciar testes com Kitchen

# Gerenciando Testes com Molecule

## Conhecendo o Molecule

## Instalar o Molecule

## Gerenciar Roles com Molecule

## Gerenciar testes com Molecule

# Gerenciar ambientes com Ansible Galaxy

## Conhecendo o Ansible Galaxy

## Comandos essenciais do Ansible Galaxy

## Enviar Roles para o Github

## Sincronizar Roles do Github com Ansible Galaxy

# Gerenciar ambientes com AWX

## Conhecer o AWX

## Instalando o AWX

## Configurar Inventory/Credential/Projetos

## Configurar Job Templates
