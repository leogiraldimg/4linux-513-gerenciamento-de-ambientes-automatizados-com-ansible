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

Troubleshooting do problema no repo com relação ao apache2 no webserver2:

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

## Roles

- Pacotes de configuração que podem ser usados para definir a configuração de um servidor
- Cada role contém um ou mais playbooks, que podem ser usados para configurar um servidor de acordo com as necessidades
- Criar um código modular, que pode ser compartilhado e reutilizado
- Ex.: role chamada "database", responsável por instalar e configurar servidores MySQL e PostgreSQL
- Encapsulando os dados necessários para realizar as tasks

### Estrutura de uma Role

- roles
  - mysql - README.md
    - defaults - main.yml
    - files
    - handlers - main.yml
    - meta - meta.yml
    - tasks - main.yml
    - templates
    - tests - inventory test.yml
    - vars - main.yml

- **defaults**: armezena o arquivo _main.yml_, contendo as variáveis que terão a prioridade mais baixa de todas as variáveis disponíveis. Pode ser facilmente substituído por qualquer outra variável, incluindo variáveis de inventário;
- **files**: armazena arquivos de configuração de cada serviço. Exemplo: haproxy.cf;
- **handlers**: armazena o arquivo _main.yml_, contendo as Handlers de cada serviço, como, por exemplo, reler ou reiniciar o HaProxy após o arquivo de configuração ser modificado;
- **meta**: armazena o arquivo _main.yml_, contendo as dependências que uma Role possui;
- **tasks**: armazena o arquivo _main.yml_, contendo as tarefas de instalação e configuração de cada serviço;
- **templates**: armazena arquivos de templates que utilizam variáveis de fatos ou personalizadas. A extensão deve terminar em .j2 com base no template Jinja2 do Python. Exemplo: _haproxy.cf.j2_;
- **tests**: o diretório tests possui um arquivo de inventário de amostra, que aponta para localhost; e um playbook _test.yml_, que está configurado para chamar a Role que você acabou de criar;
- **vars**: armazena o arquivo _main.yml_, contendo as variáveis que serão utilzadas pelo arquivo _main.yml_ da pasta tasks

### Include e Dependências

Exemplo:

- .../roles/postgresql/tasks/install.yml
- .../roles/postgresql/tasks/configure.yml
- .../roles/postgresql/tasks/service.yml
- .../roles/postgresql/tasks/main.yml

No arquivo _main.yml_:

```yaml
---
- include: install.yml
- include: configure.yml
- include: service.yml
```

São consideradas boas práticas separar a instalação de pacotes, configuração e serviços em arquivos na pasta tasks. O arquivo _main.yml_ será usado para juntar todas as configurações através da opção _include_.

O uso de dependências é comum quando uma role depende da outra. Exemplo:

- .../roles/postgresql/tasks/repositories-pgsql.yml
- .../roles/postgresql/meta/main.yml

O arquivo meta:

```yaml
---
dependencies:
  - { role: repositories-pgsql }
```

### Diretórios vars e defaults

Fornecem dados sobre suas aplicações através de Roles.

Portas, caminhos, usuários, etc.

Variáveis da pasta _defaults_ permitem fornecer valores por omissão. Esses podem ser substituídos de outros lugares, por exemplo, _vars_, _group_vars_ e _host_vars_.

Exemplo:

```yaml
# .../roles/apache/defaults/main.yml
---
apache_port: 80
```

### Roles parametrizadas

Em algumas situações, pode ser necessário substituir os parâmetros padrões especificados dentro do diretório _vars_ ou _defaults_ em uma Role.

Exemplo:

```yaml
---
- hosts: webservers
  roles:
    - { role: apache, port: 8080 }
```

### Comando ansible-galaxy

Permite criar roles (ou pacotes de roles).

Você também tem a opção de publicar suas Roles no site https://galaxy.ansible.com/ através de uma conta gratuita.

Para criar uma nova Role, use o comando _ansible-galaxy init_. Exemplo:

```console
$ ansible-galaxy init apache
```

## Gerenciar Roles no Ansible

Comando para listar as roles:

```console
$ ansible-galaxy role list
```

Para carregar uma Role, usamos a diretiva _roles_

# Gerenciando Templates no Ansible

## Templates

Arquivos de configuração com conteúdo dinâmico.

Através de Templates, precisamos apenas de um único arquivo que leva entradas dinâmicas específicas para o host que está sendo executado.

Utilizam Jinja2, que é uma linguagem de modelagem moderna para o designer que utiliza Python. Os arquivos de templates podem conter variáveis com base no template Jinja2 do Python.

Exemplo:

```python
smtpd_banner = MTA da Dexter - Servidor {% ansible_fqdn %}
```

## Tipos de Templates

Tipos de tags que os templates Jinja2 aceitam:

- **{{ }}**: incorpora variáveis dentro de um template e imprime seu valor no arquivo resultante. Este é o uso mais comum de um template;
- **{% %}**: incorpora declarações de código dentro de um template, por exemplo, para um loop. Por exemplo, as declarações if-else, que são avaliadas em tempo de execução, mas não são impressas.

**Exemplos de Tags**:

Incorporar variáveis:

```python
{{ pacotes_php }}
```

Incorporar declarações:

```python
{% ansible_default_ipv4 %}
```

## Gerenciar Roles com templates no Ansible

## Gerenciar Banco de Dados no Ansible

## Gerenciar Apache com pasta comaprtilhada por servidor NFS

# Gerenciando Testes com Kitchen

Conjunto de testes para executar código de infra em uma ou mais plataformas isoladamente.

Uma arquitetura de plug-in de driver é usada para executar código em vários provedores de nuvem e tecnologias de virtualização.

O Kitchen cria, destrói, provisiona e verifica sistemas. Os sistemas são criados usando um driver backend, os mais comuns são Vagrant e Docker. Quando os sistemas ficam disponíveis, o Kitchen pode provisioná-los por meio de um processo de convergência, usando uma solução de configuração de alteração popular como Ansible ou Chef. Após o provisionamento, você pode verificar a correção dessas alterações por meio de um plug-in verificador.

O verificador é o sistema _busser_, que instalará a ferramenta de teste no sistema local e a verificará usando a execução local no sistema convidado. Você pode usar outros plug-ins.

## Gerenciar Roles com Kitchen

Necessário instalar o *ruby-full*:

```console
$ sudo apt install ruby-full -y
```

Instale os pacotes necessários para utilizar o **kitchen**:

```console
$ sudo gem install test-kitchen kitchen-docker kitchen-ansible
```

Principais comandos do kitchen:

- **init role**: cria a estrutura de testes do kitchen em uma Role existente;
- **create**: usa o provisionador para criar as instâncias;
- **list**: lista o status das instâncias;
- **converge**: usa o provisionador Ansible para aplicar as configurações da Role nas instâncias;
- **login**: permite fazer login em uma instância;
- **setup**: permite instalar o Ruby, Bundler e Serverspec para a realização dos testes;
- **verify**: executa testes Serverspec em instâncias;
- **destroy**: usa o provisionador para destruir as intâncias;
- **test**: realiza todas as etapas (criar, convergir, configurar, verificar, destruir)

Criar a infraestrutura de arquivos e pastas com o comando init:

```console
$ sudo kitchen init -D docker -P ansible_playbook
```

Arquivo kitchen.yml

```yaml
---
driver:
  name: docker

provisioner:
  name: ansible_playbook

platforms:
  - name: ubuntu-20.04
  - name: centos-8

suites:
  - name: default
    run_list:
    attributes:
```

Descrição:

- **driver**: define o Driver responsável por criar a máquina que usaremos a nossa Role;
- **provisioner**: define o provisionador que aplica as configurações da Role na instância criado pelo Driver. Em nosso caso, usaremos o provisionador ansible_playbook;
- **platforms**: define a lista de sistemas operacionais em que vamos testa a nossa Role. Como estamos usando o Driver docker, o Kitchen criará uma imagem e container;
- **suites**: define o que queremos testar. Em nosso caso, vamos usar o Serverspec para realizar testes.

Através do Serverspec, você pode escrever testes RSpec para verificar se seus servidores estão configurados corretamente.

Mostrar o estado das máquinas configuradas a partir do _kitchen.yml_:

```console
$ sudo kitchen list
```

### Criar ambiente de testes com Kitchen

Este é o conteúdo a ser colocado no kitchen.yml da role web-balancer:

```yaml
---
driver:
  name: docker

provisioner:
  name: ansible_playbook
  hosts: all
  require_ansible_repo: true
  require_ansible_omnibus: false
  ansible_verbose: true
  ansible_verbosity: 1
  ansible_diff: true
  roles_path: ../../roles
  require_chef_for_busser: true

platforms:
  - name: Ubuntu
    driver_config:
      image: ubuntu:20.04
      platform: ubuntu

suites:
  - name: default
    verifier:
      patterns:
        - test/integration/default/serverspec/default_spec.rb
```

O caminho test/integration/default/serverspec/default_spec.rb.

Crie a instância através do subcomando _create_:

```console
$ sudo kitchen create
```

Para remover a instância, utilize o subcomando _destroy_:

```console
$ sudo kitchen destroy
```

### Testar o funcionamento de uma Role com kitchen

O arquivo default.yml será responsável por carregar a Role durante a etapa de Converge:

```yaml
---
- hosts: localhost
  roles:
    - web-balancer
```

Aplicar a etapa de convergência, para que nosso provisionador Ansible aplique as configurações de Role no container:

```console
$ sudo kitchen converge
```

Após o converge, ao listar as instâncias, perceba que o estado mudou para Converged:

```console
$ sudo kitchen list
Instance        Driver  Provisioner      Verifier  Transport  Last Action  Last Error
default-Ubuntu  Docker  AnsiblePlaybook  Busser    Ssh        Converged    <None>
```

### Logar nas instâncias de testes

Podemos logar nas instâncias através do subcomando _login_:

```console
$ sudo kitchen login default-Ubuntu
```

### Testar o funcionamento da aplicação com o Kitchen

Antes de realizar a verificação de infraestrutura, precisamos instalar o Ruby, Bundler e Serverspec com o subcomando _setup_:

```console
$ sudo kitchen setup
-----> Starting Test Kitchen (v3.2.2)
-----> Setting up <default-Ubuntu>...
       Finished setting up <default-Ubuntu> (0m0.00s).
-----> Test Kitchen is finished. (0m0.33s)
```

Antes de realizar o teste, precisamos criar a infraestrutura de diretórios que definimos no arquivo kitchen.yml:

```console
$ sudo mkdir -p test/integration/default/serverspec
```

Conteúdo do arquivo test/integration/default/serverspec/default_spec.rb:

```ruby
require 'serverspec'

set :backend, :exec

describe package('nginx') do
  it { should be_installed }
end

describe file('/etc/nginx/nginx.conf') do
  it { should contain 'proxy_pass http\:\/\/cluster' }
end

describe service('nginx') do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end
```

O arquivo default_spec.rb verificará se o pacote nginx está instalado, se o serviço nginx está sendo executado e escutando na porta 80, e se o arquivo /etc/nginx/nginx.conf possui a string _proxy_pass_ http://cluster.

Verifique a infraestrutura com o subcomando _verify_:

```console
$ sudo kitchen verify
```

### Realizar teste completo com o Kitchen

Antes de realizar o teste completo, vamos destruir a instância.

```console
$ sudo kitchen destroy
```

Vamos realizar o teste completo (criar, convergir, configurar, verificar e destruir) através do subcomando test:

```console
$ sudo kitchen test
```

[Exemplos de testes com o Serverspec](https://serverspec.org/resource_types.html)

# Gerenciando Testes com Molecule

O Molecule é um projeto que fornece uma abordagem para automatizar o desenvolvimento e teste de funções do Ansible. Ele permite que você crie uma série de testes funcionais para verificar se as suas funções estão funcionando corretamente. Além disso, também fornece um ambiente para executar esses testes com uma série de dispositivos de destino.

## Criar Roles com Molecule

Instale os pacotes necessários para utilizar o **molecule**:

```console
$ sudo pip3 install "molecule[docker,lint,ansible]"
```

Verifique os componentes presentes na infraestrutura do Molecule:

```console
$ sudo molecule --version
molecule 3.6.1 using python 3.8
    ansible:2.12.2
    delegated:3.6.1 from molecule
    docker:1.1.0 from molecule_docker requiring collections: community.docker>=1.9.1
```

Valide se o Docker está presente na lista de Drivers:

```console
$ sudo molecule drivers
╶───────────────────────────────────────────────────────────────────────────────────────────╴
  delegated
  docker
```

Crie a infra de arquivos e pastas em uma nova Role com o subcomando:

```console
$ sudo molecule init role acme.apache --driver-name docker
```

Função dos arquivos em apache/molecule/default:

- **converge.yml**: contém a chamada para a sua Role. O Molecule invocará este playbook através do ansible-playbook e o executará em uma instância criada pelo driver;
- **molecule.yml**: arquivo principal do Molecule, responsável por configurar o ambiente de testes definindo drivers, provisionadores e ferramentas de verificação;
- **verify.yml**: contém testes específicos em relação ao estado do contêiner, sendo o arquivo que o Ansible usa como verificador padrão.

Conteúdo do arquivo _molecule.yml_:

```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: Ubuntu
    image: geerlingguy/docker-ubuntu2004-ansible
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    pre_build_image: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
  - name: CentOS
    image: geerlingguy/docker-centos8-ansible
    pre_build_image: true
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
provisioner:
  name: ansible
verifier:
  name: ansible
lint: |
  set -e
  yamllint .
  ansible-lint .
```

- **dependency**: define as dependências da Role;
- **driver**: define o Driver responsável por criar a máquina que usaremos para testar a nossa Role;
- **platforms**: define a lista de sistemas operacionais em que vamos testar a nossa Role. Como estamos usando o driver Docker, podemos personalizar informando imagens de repos públicos ou privados;
- **provisioner**: define o provisionador que aplica as configurações da Role, instância criada pelo Driver. Em nosso caso, usaremos o provisionador ansible_playbook;
- **verifier**: define a ferramenta de verificação. Em nosso caso, usaremos o Ansible.

Estamos usando o Lint para verificar a sintaxe de nossas Roles

Mostrar o estado de todas as máquinas configuradas a partir do arquivo _molecule.yml_:

```console
$ sudo molecule list
INFO     Running default > list
                ╷             ╷                  ╷               ╷         ╷
  Instance Name │ Driver Name │ Provisioner Name │ Scenario Name │ Created │ Converged
╶───────────────┼─────────────┼──────────────────┼───────────────┼─────────┼───────────╴
  Ubuntu        │ docker      │ ansible          │ default       │ false   │ false
  CentOS        │ docker      │ ansible          │ default       │ false   │ false
                ╵             ╵                  ╵               ╵         ╵
```

Instalar o _ansible-lint_:

```console
$ sudo apt install ansible-lint -y
```

## Verificar sintaxe de playbooks com Molecule

Para testar a sintaxe dos playbooks, execute o subcomando _lint_:

```console
$ sudo molecule lint
```

## Criar ambiente de testes com Molecule

Crie a instância através do subcomando _create_:

```console
$ sudo molecule create
```

Verifique se as instâncias possuem o status _true_ na coluna Created:

```console
$ sudo molecule list
INFO     Running default > list
                ╷             ╷                  ╷               ╷         ╷
  Instance Name │ Driver Name │ Provisioner Name │ Scenario Name │ Created │ Converged
╶───────────────┼─────────────┼──────────────────┼───────────────┼─────────┼───────────╴
  Ubuntu        │ docker      │ ansible          │ default       │ false   │ false
  CentOS        │ docker      │ ansible          │ default       │ false   │ false
                ╵             ╵                  ╵               ╵         ╵
```

É possível verificar que novas imagens do Docker foram criadas:

```console
$ sudo docker image list
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
<none>       <none>    26cfe63082ec   22 hours ago   233MB
ubuntu       20.04     54c9d81cbb44   2 weeks ago    72.8MB
```

Para remover a instância, utilize o subcomando _destroy_:

```console
$ sudo molecule destroy
```

É possível verificar que os containers não estão mais em execução, mas as imagens permanecem:

```console
$ sudo docker container list

$ sudo docker image list
```

## Adicionar estrutura do Molecule em uma role existente

Para começar, vamos acessar o diretório da Role nfs-server:

```console
$ sudo molecule init scenario -r nfs-server -d docker
```

Arquivo _molecule.yml_:

```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: Ubuntu
    image: docker-nfs-server-ubuntu:latest
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    pre_build_image: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
  - name: CentOS
    image: docker-nfs-server-centos:latest
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    pre_build_image: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
provisioner:
  name: ansible
verifier:
  name: ansible
lint: |
  set -e
  yamllint .
  ansible-lint .
```

Foi possível observar que o arquivo _molecule.yml_ está usando as imagens _docker-nfs-server-ubuntu:latest_ e _docker-nfs-server-centos:latest_. Essas imagens não estão disponíveis no Docker Hub, sendo assim, é necessário criá-las;

Para criá-las, utilizamos o arquivo modelo _Dockerfile_ da pasta docker-nfs-server-ubuntu, disponível na HOME do usuário suporte:

```dockerfile
FROM geerlingguy/docker-ubuntu2004-ansible
ENV DEBIAN_FRONTEND noninteractive
RUN apt update -qq && apt install -y nfs-kernel-server runit inotify-tools -qq
RUN mkdir -p /opt/site
VOLUME /opt/site
EXPOSE 111/udp 2049/tcp
```

Para gerar uma nova imagem Docker use o comando _docker image build_:

```console
$ sudo docker image build -t docker-nfs-server-ubuntu ~/docker-nfs-server-ubuntu
```

Para verificar se a imagem foi gerada:

```console
$ sudo docker image ls docker-nfs-server-ubuntu
```

## Testar o funcionamento da Role com Molecule

Vamos aplicar a etapa de convergência para que nosso provisionador Ansible aplique as configurações da Role no container:

```console
$ sudo molecule converge
```

## Testar a idempotência de uma Role nfs-server com Molecule

Para testar a idempotência em nossa Role, execute o subcomando _idempotence_:

```console
$ sudo molecule idempotence
```

## Testar o funcionamento da aplicação com Molecule

Arquivo _verify.yml_:

```yaml
---
- name: Verifica o funcionamento do servidor NFS
  hosts: all
  gather_facts: false
  tasks:
  - name: Example assertion
    assert:
      that: true
  - name: Testando o compartilhamento NFS
    shell: "showmount -e --no-headers"
    register: showmount_result
    failed_when: showmount_result.stdout != "/opt/site 172.16.0.0/24"
```

Para verificar se a aplicação está funcionando corretamente utilize o subcomando _verify_:

```console
$ sudo molecule verify
```

## Realizar teste completo na role

Teste completo = dependência, lint, limpeza, destruição, sintaxe, criação, preparação, convergência, idempotência, efeito lateral, verificação e limpeza.

```console
$ sudo molecule test
```

# Gerenciar ambientes com Ansible Galaxy

## Ansible Galaxy

Repositório de pacotes de software para Ansible. Ele permite baixar e usar pacotes de software de automação de infraestrutura construídos por outros usuários da comunidade Ansible. Os pacotes de software estão disponíveis para todos os sistemas operacionais suportados por Ansible.

Para usar o Galaxy, você precisa ter uma conta gratuita na página do [Galaxy](https://galaxy.ansible.com)

## Comandos essenciais do Ansible Galaxy

```console
$ sudo ansible-galaxy role list
$ sudo ansible-galaxy role search ntpdate
$ sudo ansible-galaxy role search --author geerlingguy
$ sudo ansible-galaxy role search --author geerlingguy --galaxy-tags ntp
$ sudo ansible-galaxy role info geerlingguy.ntp
$ sudo ansible-galaxy role install geerlingguy.ntp
$ sudo ansible-galaxy role remove geerlingguy.ntp
```

## Sincronizar Roles no Github com Ansible Galaxy


# Gerenciar ambientes com AWX

## Conhecer o AWX

## Instalando o AWX

## Configurar Inventory/Credential/Projetos

## Configurar Job Templates
