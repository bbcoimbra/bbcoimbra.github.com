# Automação de ambientes de Infra com Puppet

!

### Bruno Coimbra (@bbcoimbra)
* 1/2 Dev 1/2 Sysadmin
* Ex-Fatecano
* Rockabilly Addict
* Apreciador de Café e Cerveja

!

## O que é?

Um framework e engine de automatização de adminstração para ambientes de infra.
(Há suporte parcial para ambientes Windows.)

!

### O que é?

Possui a capacidade de provisionar e manter o estado das configurações das máquinas.

!

### O que é?

Foi desenvolvido pela PuppetLabs. É utilizado por empresas com grandes ambientes de Infra:

+ Sun Microsystem (Case publicado)
+ Zynga (Case publicado)
+ Twitter
+ Rackspace

!

## Dependências

  + Ruby (compilado com suporte para openssl)
  + gem Facter

!

## Facter

    $ facter
    architecture => x86_64
    hostname => asgard
    interfaces => eth0,lo,wlan0
    memorytotal => 3.61 GB
    ps => ps -ef
    rubyversion => 1.8.7
    uptime => 2:15 hours
    (... e mais uma penca de fatos ...)

!

## Funcionamento

Os nós agentes, de tempos em tempos, enviam um conjunto de pares chave-valor (fatos) para
o nó master, este analisa os dados recebidos e compila um *catálogo* com aquilo que deve
existir no nó agente e o envia.

!

## Funcionamento
O nó agente verifica as alterações que precisam ser aplicadas e executa-as.

!

## Instalação

Debian

    apt-get install puppet
    apt-get install puppetmaster

!

## Instalação

Como gem

    gem install puppet

!

# Configuração

!

### Configuração

* Master / Agents

Configuração depende de um servidor central que recebe requisição dos agentes 
por uma conexão segura e toda autenticação é feita com troca de certificados
(por HTTPS).

!

### Master

Para a configuração inicial do nó master é necessário apenas os
seguintes arquivos:

    /etc/puppet/
    |-puppet.conf
    |-manifests/
    |--site.pp

!

### Master

    $ cat puppet.conf
    [master]
    certname=asgard.bbcoimbra.com

!

### Master

Inicialmente o arquivo /etc/puppet/manifests/site.pp pode estar vazio, embora
sua presença seja obrigatória.

!

### Master

    $ puppet master --no-daemon

!

### Agents

    $ cat /etc/puppet/puppet.conf
    [agent]
    server = puppet.bbcoimbra.com

!

### Agents

    $ puppet agent -vt
    info: Creating a new SSL key for asgard
    info: Creating a new SSL certificate
    request for asgard
    info: Certificate Request fingerprint
    (md5): <MD5 HASH>
!

### Master

    $ puppet cert --list
    asgard <MD5 HASH>

!

### Master

    $ puppet cert --sign asgard
    notice: Signed certificate request
    for asgard
    notice: Removing file
    Puppet::SSL::CertificateRequest asgard
    at '/etc/puppet/ssl/ca/requests/asgard.pem'

!

### Agent

    $ puppet agent -vt
    info: Caching certificate for asgard
    info: Caching catalog for asgard
    info: Applying configuration
    version '1341370642'
    info: Creating state file
    /var/lib/puppet/state/state.yaml
    notice: Finished catalog run
    in 0.03 seconds

!

# Manifestos

!

### Manifestos

Nome dado aos "programas" do Puppet e possuem extensão *.pp* 

    $ cat manifests/site.pp
    node 'asgard.bbcoimbra.com' {
      package { 'vim':
        ensure => present,
      }
    }

!

### Manifestos

Contém:

* Nós - Hosts que rodam os agentes
* Classes - Conjuntos de recursos
* Recursos
* Definições - Define objetos reusáveis
* Módulos - Conjuntos de Classes, Definições, Templates

!

#### Nós

    node 'asgard.bbcoimbra.com' {
      include default_node
    }

!

#### Nós

    node 'www.test.com' inherits default_node {
      include ruby1.8.7
    }

!

#### Classes

    class default_node {
      user {'bruno':
        home => '/home/bruno',
        shell => '/bin/bash',
      }
      package { 'vim':
        ensure => present,
      }
    }

!

#### Classes

    class ruby1.8.7 {
      include default_node
      package { 'ruby1.8':
        ensure => present,
      }
    }

!

## Recursos

+ Package
+ File
+ User
+ Service
+ Exec
+ ...

!

### Recursos

    package { 'vim':
      ensure => present,
    }

!

### Recursos

    file { "/etc/passwd":
      owner => root,
      group => root,
      mode => 644
    }

!


### Recursos

    file { "/etc/nginx/nginx.conf":
      owner => root,
      group => root,
      mode => 644,
      source => 'puppet:///nginx/nginx.conf'
    }

!

### Recursos

    user { "bruno":
      home => '/home/bruno',
      group => 'users',
      shell => '/bin/bash'
      ensure => present,
    }

!

### Recursos

    service { 'nginx':
      ensure => running,
      enable => true,
      hasrestart => true,
      require => File['/etc/nginx/nginx.conf'],
      restart => '/etc/init.d/nginx reload'
    }

!

### Recursos

    exec {'/usr/bin/git fetch':
      cwd => '/srv/my_git_project',
    }

!

### Recursos

    exec {'/usr/bin/git clone <git_repo>':
      cwd => '/srv/',
      creates => '/srv/<git_repo>/.git'
    }

!

## Definições de novos Tipos

São utilizadas para reduzir redundância na
declaração de recursos

!

### Definições

Suponhamos que controlamos os usuários de um servidor git
(gitolite, por exemplo) com o seguinte recurso:

    user { "bruno":
      home => '/home/git',
      group => 'git',
      shell => '/usr/bin/git-shell'
      ensure => present,
    }

!

### Definições

    define git_user($user) {
      user { $user:
        home => '/home/git/$user',
        group => 'git',
        shell => '/usr/bin/git-shell'
        ensure => present,
      }
    }

!

### Definições

    class git_users {
      git_user {'bbcoimbra'},
      git_user {'bruno'},
    }
!

## Módulos

São conjuntos completos de manifestos auto-contidos e reusáveis.

    /etc/puppet/
    |-modules/
    |--nginx/
    |---manifests/
    |----init.pp

!

### Módulos

São carregados automáticamente ficam disponíveis para utilizar
nos manifestos.

!

### Módulos

No arquivo *init.pp* é definida a *classe* que nomeia o módulo.
Os outros arquivos e diretórios seguem a mesma estrutura do 
diretório de confs do puppet **/etc/puppet/{files,manifests,templates}/**.

!

### Módulos

São incluidos nos arquivos de manifestos como qualquer outra classe com
a instrução *include*

!

### Módulos

Há vários módulos disponíveis no Puppet Forge e no GitHub

* http://forge.puppetlabs.com/

!

## Testes

cucumber-puppet

!

## Pontos de atenção

!

### Pontos de atenção

Plano de Rollback bem definido

!

### Pontos de atenção

Cuidado com os deploys automatizados

!

### Pontos de atenção

Definição cuidadosa da arquitetura
(Não confundir recursos específicos com genéricos)

!

## Outras Funcionalidades

Extender Facter, Definir funções e providers.

!

# Dúvidas?

!

## Recursos

* #puppet@freenode.net (700+ usuários)
* Livro Pró Puppet
* http://docs.puppetlabs.com/
* http://forge.puppetlabs.com/

!

# Obrigado!

!

# Contatos

+ Bruno Coimbra
+ bbcoimbra@gmail.com
+ github.com/bbcoimbra
+ @bbcoimbra
+ facebook.com/bbcoimbra

