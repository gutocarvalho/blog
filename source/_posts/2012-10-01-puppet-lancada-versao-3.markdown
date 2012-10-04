---
layout: post
title: "Puppet: Lançada versão 3.0"
date: 2012-10-01 14:11
comments: true
categories: [ puppet, tecnologia ]
---
Foi lançado no dia 28/Set/12 a versão [3.0](http://docs.puppetlabs.com/puppet/3/reference/release_notes.html) do Puppet

## Instruções para instalação

Você pode fazer o download do código fonte em<br>
[http://downloads.puppetlabs.com/puppet/puppet-3.0.0.tar.gz](http://downloads.puppetlabs.com/puppet/puppet-3.0.0.tar.gz)

Instruções para instalar via repositórios puppetlabs (yum/apt)<br>
[http://docs.puppetlabs.com/guides/puppetlabs_package_repositories.html](http://docs.puppetlabs.com/guides/puppetlabs_package_repositories.html)

Você poder fazer o download da gem no rubygem<br>
[https://rubygems.org/downloads/puppet-3.0.0.gem](https://rubygems.org/downloads/puppet-3.0.0.gem)<br>
ou se preferir pode instalar usando o comando: `gem install puppet`

Você pode fazer o download do pacote para OSX:<br>
[http://downloads.puppetlabs.com/mac/puppet-3.0.0.dmg](http://downloads.puppetlabs.com/mac/puppet-3.0.0.dmg)

Você pode fazer o download do pacote para Windows:<br>
[http://downloads.puppetlabs.com/windows/puppet-3.0.0.msi](http://downloads.puppetlabs.com/windows/puppet-3.0.0.msi)

Se você usa `ensure => latest` para o pacote do puppet, a partir do dia 01 de Outubro ele já estará nos repositórios oficiais e será então instalado automaticamente.

## Novidades

* Performance Improvements - substantial improvements to performance,
particularly around catalog compilation. Agent now uses JSON for
catalog cache, which can be dramatically faster for large catalogs
(#16058, #2892)
* Data Bindings - hiera will be automatically consulted for values of
parameterized classes so you don't need the parser functions. (#11608)
* Improved OS/Platform Support - Full Ruby 1.9 support; vastly
improved Windows package support; yumrepo now supports ssl options
(#3324); better upstart support; better Solaris zone, package and
service support;
* Loading Plugins from Rubygems - you can now install and use puppet
extension code (faces, types, providers) via rubygems. (#7788)
* Server Auto-Discovery - Puppet agents can use SRV records in DNS to
find CA, master, report, and file servers (#3669)
* Semantic Versioning - With 3.0.0, Puppet Labs makes a commitment to
follow the Semantic Versioning guidelines outlined on semver.org.
Public, documented APIs won't break until 4.0; minor-version 3.x
releases will add new features while retaining compatibility, and
tiny-version 3.0.x releases will be bugfix-only.

## Mudanças em DSL/Config

* Variable Scoping - Dynamic scoping has been removed.
http://docs.puppetlabs.com/guides/scope_and_puppet.html
* Auth.conf differentiates between names and IPs - There's a new
`allow_ip` keyword in auth.conf if you want to permit IP addresses.
(PR991)
* `unless` is available as a synonym for `if !` (#7762)
* Pluginsync now defaults to true - Yep, it's true. (#5521)
* Updated `config.ru` syntax -- When running the master under Rack,
make sure you update your config.ru to follow the example from
ext/rack/files/config.ru in the source distribution. (#15337)

## Changelog

Para saber mais informações sobre o lançamento, acesse a mensagem de divulgação na lista [puppet-announce](https://groups.google.com/group/puppet-announce/browse_thread/thread/96a993057f570edc).

[s]<br>
Guto
