---
layout: post
title: "O que é DevOps afinal?"
date: 2013-03-16 15:47
comments: true
categories: tecnologia
---

{% img /images/posts/bluedevops.png [Blue DevOps] %}

## 1. Papo sobre DevOps


Hoje vim falar sobre a buzzword **DevOPs**. Algumas pessoas tem me procurado pelo blog para conversar sobre este termo, e normalmente eles querem que eu responda as seguintes perguntas:

1. O que significa DevOps?
2. DevOps é um movimento?
4. DevOps é uma filosofia, é um conceito ou uma cultura?
5. DevOps é uma metodologia?
6. DevOps é algum tipo de ambiente ou grupo de ferramentas ?
6. O especialista DevOps é um devel que entende de infra?
7. O especialista DevOps é um sysadmin que entende de devel?
8. DevOps é um cargo? é um setor ou um departamento?
9. DevOps só funciona em startups ou serve para o ambiente corporativo?
10. O DevOps é algo novo?

Ao longo do texto eu pretendo responder a todas estas questões, mas para respondê-las vamos precisar entender algumas coisas antes.

## 2. Como tudo começou?

Para que possamos compreender de forma plena, precisamos ir ao cerne desta história. O movimento DevOPs não começou em um lugar só, existem muitos lugares que dão pistas sobre as origens do termo, mas aparentemente as informações mais [concretas](http://itrevolution.com/the-convergence-of-devops/) sobre as origens deste movimento nos levam ao ano de 2008. Neste ano, começaram a utilizar o termo **infraestrutura ágil** em algumas listas de discussão com foco em desenvolvimento ágil, e na mesma época durante evento **Agile 2008**, surgiram conversas que abordavam o tema metodologia ágil para a administração de infraestrutura, inspirada no modelo ágil de desenvolvimento, no entanto, foi a lista de discussão européia com nome **agile-sysadmin** que começou a abordar o tema com propriedade, com isso ela ajudou a colocar os primeiros tijolos na ponte que faria a ligação entre developers e sysadmins. Um dos participantes desta lista era Patrick Debois (@patrickdebois), além de muito ativo ele era e ainda é um grande entusiasta do assunto.

O termo DevOPS só foi criado de fato em 2009 durante a conferência Velocity da O'Reilly, nesta conferência John Allspaw (Etsy.com) e Paul Hammond (Typekit) apresentaram o trabalho **10+ Deploys Per Day: Dev and Ops Cooperation at Flickr**, veja abaixo os slides desta palestra.

<center><iframe src="http://www.slideshare.net/slideshow/embed_code/1628368?rel=0" width="597" height="486" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen webkitallowfullscreen mozallowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="http://www.slideshare.net/jallspaw/10-deploys-per-day-dev-and-ops-cooperation-at-flickr" title="10+ Deploys Per Day: Dev and Ops Cooperation at Flickr" target="_blank">10+ Deploys Per Day: Dev and Ops Cooperation at Flickr</a> </strong> from <strong><a href="http://www.slideshare.net/jallspaw" target="_blank">John Allspaw</a></strong> </div></center><br>

A palestra acima deixou Patrick Debois muito animado, foi então que ele teve a ideia de criar um encontro chamado DevOpsDay, este encontro ocorreu em Ghent no final de 2009, foi um encontro de dois dias e aparentemente foi lá que tudo começou.

De lá para cá, Patrick Debois, Gildas Le Nadan, Andrew Clay Shafer, Kris Buytaert, JezzHumble, Lindsay Holmwood, John Willis, Chris Read, Julian Simpson, R.I.Piennar (mcollective) e muitos outros levaram o evento para diversas localidades, dentre elas:

* New York 2012
* Rome 2012
* Mountain View 2012
* India 2012
* Tokyo 2012
* Austin 2012
* Goteborg 2011
* Bangalore 2011
* Melbourne 2011
* Mountain View 2011
* Boston 2011
* Göteborg 2011
* Sao Paulo 2010
* Hamburg 2010
* Mountain View 2010 (video intro)
* Sydney 2010
* Ghent 2009
 
  
É importante falar que ao levar o evento para diversos países, estas pessoas foram responsáveis por disseminar
a cultura DevOps pelo globo, com isso, direta ou indiretamente eles se tornaram a força motriz de uma discreta revolução 
no mundo da TI. 

Inicialmente a cultura DevOps se mostrou muito presente no ambiente das startups,  porém, algum tempo depois começou a fazer parte do mundo corporativo, aqui neste texto procuro trazer a visão da cultura DevOps no meio corporativo.

Na abertura do evento DevOpsDay há sempre um vídeo de intro, veja dois deles:

Ghent 2009

<iframe width="420" height="315" src="http://www.youtube.com/embed/EOveXZhJpr4" frameborder="0" allowfullscreen></iframe>

Mountain View 2010

<iframe width="560" height="315" src="http://www.youtube.com/embed/a0N2ugDwi5g" frameborder="0" allowfullscreen></iframe>

## 3. DevOps Manifest

Apesar de terem organizado o DevOpsDays em diversos países, não foi estabelecido um manifesto para o assunto, logo existem muitas interpretações acerca deste termo.

Mas antes de argumentar acerca do possível conteúdo de um manifesto DevOps, primeiro temos que entender a dinâmica na relação entre as áreas de infraestrutura (infra) e desenvolvimento (devel).

## 4. Analisando Infra e Devel

Para entender melhor o que DevOps significa, precisamos então analisar de forma prática e direta a vida de sysadmins, desenvolvedores e o cotidiano destas áreas, eu espero que isto lhe ajude a compreender melhor o assunto adiante.

Vamos imaginar - hipoteticamente - uma empresa de comunicação que desenvolve aplicações web em sua maioria para portais de notícias, e em alguns casos também faz aplicações web internas (rh, financeiro, administrativo), nessa empresa o devel trabalha com PHP, PYTHON, RUBY e JAVA.

Para um melhor entendimento, considere as duas características abaixo como cotidianas nesta empresa fictícia:

1. O Devel está começando a trabalhar com metodologias ágeis (pró-ativo, evolutivo e contínuo).

2. A Infra continua trabalhando no modelo tradicional de administração (manual, caótico e reativo).

### 4.1 Infra em foco

A infra é composta em parte pelos sysadmins, estes rapazes e moças tem a missão de manter os sistemas funcionando, são eles que fazem os deploys e os rollbacks das aplicações do devel, é responsabilidade deles manter o ambiente de produção intacto.

Os sysadmins tem que rodar as aplicações, monitorar o funcionamento, a performance, avaliar e propor melhorias de forma a manter as aplicações sob seu cuidado a pleno vapor - rodando de forma rápida e estável, além disto, eles devem planejar as mudanças com cautela, tentando minimizar os riscos envolvidos.

Eles se preocupam com segurança, estabilidade e principalmente com o acordo de nível de serviço (SLA) de cada produto sob sua responsabilidade, esta preocupação é fundamental para o negócio.

Entenda que se uma aplicação parar de funcionar isto vai impactar no SLA, podendo significar perdas financeiras significativas relacionadas ao produto, afinal produto fora significa cliente insatisfeito e prejuízo, seja ele financeiro, seja ele institucional, e no caso da sustentação do produto, isto significa que a prestadora do serviço, neste caso a empregadora dos sysadmins, pode ser multada. De toda o jeito, o que ocorre de forma direta é diminuição do valor do negócio.

Em resumo, a infra (sysadmins) se preocupa em **proteger o valor do négocio**.

### 4.2 Devel em foco

O devel é composto em parte por desenvolvedores, estas moças e rapazes trabalham com lógica e criatividade, eles passam boa parte de seu tempo codificando soluções, e focam seu trabalho nos requisitos que o analista conseguiu mapear junto ao cliente.

Os desenvolvedores estão constantemente criando e aprimorando suas aplicações, com isto novas versões são criadas e precisam ser disponibilizadas, assim seus clientes poderão usufruir dos recursos solicitados.

Nova versão significa novo deploy, e caso ocorra algum problema, isto irá demandar um rollback, ambos procedimentos envolvem equipes de infra.

Em resumo, podemos dizer que o devel se preocupa em **aumentar o valor do negócio**.

### 4.3 Onde está o conflito?

{% img /images/posts/relacao-infra-devel.png [Relação Infra e Devel] %}

Os desenvolvedores querem colocar suas aplicações no ar o mais rápido possível, no entanto os sysadmins querem ter certeza que a aplicação está
estável o suficiente para entrar em produção sem gerar incidentes.

Nos últimos anos esse conflito foi latente no mundo de TI, algumas empresas tinham regras tão rígidas que só permitiam deploy uma vez por semana - em casos mais rígidos apenas uma vez por mês, tudo isto pensando em proteger o negócio.

Com o devel mudando de metogologia, nem preciso dizer que esse método de deploy - uma vez por semana - não combina com desenvolvimento ágil, com isso a infra teve que evoluir a fórceps, e quem antes fazia deploy uma vez por semana, teve que aprender a fazer várias vezes por dia.

É claro que a infra trabalhando com os métodos que estava acostumada (deploy 1 vez por semana e manual) não dava vazão as demandas, e também é óbvio que o devel não possuía uma infra adequada para fazer o desenvolvimento de forma contínua. 

Além de tudo isso, normalmente o devel não conhece e não tem como prever aspectos importantes relativos a infra que fica de cara para o cliente, portanto, quando a aplicação vai para produção, normalmente ocorrem - constantes - pequenos incidentes que geram uma enorme perda de valor no negócio. Traduzindo, são aqueles ajustes na aplicação que precisam ser feitos de última hora pois o ambiente devel é completamente diferente da produção.

O cliente por sua vez reclama - com razão - e depois a gerência de TI ficava tentando encontrar o dono do problema (caça as bruxas), de um lado devel dizendo que infra é engessada, lenta e que não oferece um ambiente adequado para desenvolverem suas aplicações, do outro lado a infra dizendo que o devel faz código ruim e instável e que não é culpa deles se a aplicação não funciona.

Eu sou de infra há muitos anos, mas tenho que dizer que a infra devido a culturas arcaicas de administração, heranças do tempo dos mainframes, tem mais culpa no cartório neste cenário, porém o devel também tem seus problemas, afinal, como estão começando a aplicar métodos ágeis, ainda estão criando a cultura de execução de testes e garantia de qualidade (QA).

#### 4.3.1 Incidentes

Quando ocorre algum incidente, você vai ouvir da infra falando para o devel que **o problema não são as máquinas, é o código**, e certamente o devel vai falar para infra que **o problema não é o código, são as máquinas**, e provavelmente ainda vão dizer que o sistema **está funcionando no  notebook deles**, e infelizmente isso será algo cotidiano. 

{% img right /images/posts/opsburn.png [Ops Burn] %}

Espero que neste ponto você já esteja enxergando o problema.

É preciso entender que infra e o devel trabalham em nichos, cada um no seu quadrado, cada um em sua realidade e nenhum deles está muito disposto a mudar sua cultura, nenhum deles está disposto a ceder. A infra não conhece o devel e não sabe como mudar para ajudá-los, o devel não conhece a infra e não sabe como pedir o que precisam.

No final das contas, as pessoas não conseguem estabelecer uma forma sadia e eficiente de comunicação, e com isso, não existe trabalho colaborativo entre estas duas áreas.

### 4.4 O combustível do conflito

{% img /images/posts/no.png [Combustivel do Conflito No] %}

Acima eu apresentei o conflito comum entre as áreas, porém existe o combustível que o mantém acesso, e esse combustível é o comportamento do sysadmin, portanto, há de fato uma razão para se ter tanto ódio deles, vamos a elas:

* Eles falam não
* Eles falam não pela segunda vez
* Eles falam não pela terceira vez
* Eles falam não o tempo todo por diversas razões, para diversos pedidos
* Eles demoram, atrasam e perdem prazos de atendimento
* Eles se recusam a quebrar as coisas mesmo que seja para encontrar o problema
* Eles se preocupam com UPTIME e não com o negócio
* Eles acham que o devel só quer saber de perfumarias e coisas do gênero
* Eles não se esforçam para ajudar o devel a encontrar o problema
* Eles acham que o problema do devel não é problema deles
* Eles não conseguem enxergar o negócio e não enxergam que infra e devel são parte de um todo

E isso tudo faz parte daquele comportamento arcaico que eu já mencionei, eu quis reforçar isto pois é bom mostrar as raizes de um problema para ajudar na reflexão do que é preciso mudar. 

Veja que tal comportamento é inaceitável e incompatível com o mundo de hoje, mesmo assim ainda é muito comum encontrar pessoas que possuem este
tipo de atitude e perfil.

### 4.5 Uma pitada de realidade

Lembra que eu disse que a infra se preocupa em **proteger o negócio** e o devel se preocupa com as **formas de agregar valor ao negócio**? 

Esqueça o que eu disse, isso funcionava nos anos 70/80/90, mas hoje isso não é suficiente.

A infra deve entender que sua obrigação é oferecer os meios para fazer o negócio fluir, e isso também é papel do devel.

Ambas equipes precisam mudar a forma de pensar e de agir, porém é preciso ter consciência de que mudanças estão associadas a problemas, uma mudança pode quebrar seu produto e afetar o seu negócio.

Então qual é a receita mágica? 

Como mudar sem afetar o negócio?

### 4.6 Mudanças necessárias

A infra previsa evoluir, e precisa fazer isto rapidamente.

O devel predisa ter controle de todas as fases do deploy.

A infra precisa começar a trabalhar de forma automatizada e dinâmica, precisa ser mais veloz para subir novos ambientes ou mesmo reconstruir/duplicar os ambientes existentes para suprir as necessidades do devel, não dá mais para trabalhar de forma manual e usar as mesmas metodologias da época dos mainframes.

O devel precisa conseguir passar para infra suas necessidades de forma clara, e tem que se esforçar para fazer a infra entender isto - e eles não vão entender na primeira vez. 

### 4.6.1 Busca de soluções

E foi a busca de soluções para estas necessidades que motivou importantes discussões no mundo da TI, foi então que começaram a falar de 'Infraesturura ágil' no ano de 2008, vamos agora entender o que é isso.

## 5. Infraestrutura ágil

### 5.1 Infraestrutua como código

A discussão acerca de infraestrutura ágil ganhou força com o crescimento de duas tendências, são elas **virtualization** e **cloud computing**. Desde 2003 empresas começaram a conviver com ambientes virtualizados, logo um parque com poucas máquinas físicas poderia se tornar um parque com dezenas máquinas virtuais, e após o recente  advento da Cloud, dezenas de máquinas virtuais podem se tornar centenas ou milhares de instâncias a serem administradas na nuvem.

Não havia mais espaço para se trabalhar infraestrutrua da forma tradicional, foi necessário dar um passo a diante e pensar em infraestrutura como código, principalmente quando paramos para analisar o recente boom das startups, empresas pequenas com produtos de enorme alcance, produtos que rodam em centenas de instâncias na nuvem, atendendo milhões de usuários, e tudo isso é administrado por equipes mínimas.

O objetivo é fazer deploy não só de aplicações, mas deploy de infraestrutura de forma rápida e controlada.

Isso significa que se você precisa subir um ambiente JBOSS você vai fazer isto em poucos minutos e não em dias.

### 5.2 Ferramentas de infraestrutura ágil

Quando se fala em infraestrutura ágil o que vem a mente são ferramentas, e de fato elas são parte disto. 

Basicamente temos três tipos de ferramentas, são elas:

1. Orquestradores
2. Ferramentas para gerenciamento de configurações
3. Ferramentas para bootstrapping e provisionamento

Orquestradores são ferramentas que nos permitem executar comandos e controlar nodes/instâncias de nosso parque em tempo real. Alguns destes são Fabric,
Capistano, Func e Mcollective.

Ferramentas de gerência de configuração normalmente controlam estados de seu sistema, ajudam a centralizar toda as configurações e facilitam a administração e criação de novos ambientes. Algumas delas são Puppet, Chef, Cfegine e Salt.

Ferramentas de bootstrapping são aquelas que nos ajudam a instalar um sistema operacional seja em uma máquina física, seja em um máquina virtual, seja em uma instância na nuvem, dentre elas temos alguns provedores de CLOUD como AWS e Rackspace que já oferecem isso nativamente, existem também ferramentas como o Kickstart e Cobbler que atuam neste segmento.

A combinação destes três tipos de ferramentas nos permite ter o que chamamos de infraestrutura ágil.

Mas apesar da qualidade e dos ganhos em utilizar tais tecnologias, a ferramenta sozinha não resolverá seus problemas, é preciso mudar a cultura e a forma de trabalhar a infraestrutura.

### 5.3 Equipe de infraestrutura ágil

Equipes que trabalham com infraestrutura ágil também precisam de um método diferenciado de organização, normalmente estas equipes estão trabalhando seguindo estes eixos:

1. Versionamento do código e arquivos de configuração (git)
2. Organização de atividades de forma visual (KANBAN BOARD)
3. Trabalho em pares
4. Divisão das atividades em sprints
5. Reuniões ágeis diárias (standup meeting de 10 minutos - em pé)
6. Reuniões ágeis periódicas (retrospectiva e planejamento de sprints).

Parece com SCRUM mas não é, mas é fortemente inspirado nele e no Kanban.

### 5.4 Então DevOps e Infraestrutura ágil são a mesma coisa?

Não, infraestrutura ágil é parte da cultura DevOps.

DevOps depende de infraestutura ágil, mas ainda tem muito mais.

Apesar da evolução da infra, ainda falta algo que conecte as duas áreas, precisamos de um agente de mudanças principalmente
no meio corporativo.

## 6. Cultura DevOps

Chegou a hora de entrar neste assunto, agora nós vamos aprofundar nossos estudos em relação a cultura DevOps.

### 6.1 Características da cultura DevOps

Para entender a cultura devops sem fazer um texto muito longo, vou pontuar as suas principais características - em minha opinião.

#### 6.1.1 Em relação as características da cultura

Patrick Debois (foi quem cunhou o termo) diz que DevOPs essencialmente é uma cultura, e a descreve utilizando 4 eixos, são eles:

* Cultura
    * Colaboração
    * Fim das divisões
    * Relação saudável entre as áreas
    * Mudança de comportamento
* Automação 
    * Deploy
    * Controle
    * Monitoração
    * Gerência de configuração
    * Orquestração
* Avaliação
    * Métricas
    * Medições
    * Performance
    * Logs e integração
* Compartilhamento
    * O feedback é tudo
    * Boa comunicação entre a equipe

Eu prentendo detalhar um pouco mais estes eixos, vamos lá.

#### 6.1.2 Em relação as características técnicas

Um ambiente DevOps deve ter/possuir/oferecer/permitir:

* Infraestrutura como código
* Orquestração de servidores
* Gerência de configurações
* Provisionamento dinâmico de ambientes
* Controle de versões compartilhado entre infra e devel
* Ambiente de desenvolvimento, teste e produção (no mínimo)
* O ambiente de devel deve possibilitar TDD
* Infra deve participar dos projetos desde o início [1]
* Infra deve participar das reuniões de devel [2]
* Devel deve participar das reuniões de infra [3]
* Ambiente de entrega contínua [4]
* Os desenvolvedores devem conseguir fazer o deploy sem interferência da infra [5]
* Monitoramento eficaz com processamento adequado dos eventos e métricas
* Capacidade de resposta rápida a incidentes e problemas
* Backup e restore confiável

[1] O devel precisa envolver a infra nos projetos desde o início - isso significa participar das reuniões técnicas ou SCRUM, afinal sem a infra não há projeto, e além disto, quanto mais problemas foram resolvidos durante o projeto - com ajuda da infra, menos problemas serão expostos aos clientes. 

[2] A infra também precisa observar quais são as metas da empresa a longo prazo, principalmente aquelas ligadas ao devel, pois enxergando onde o devel quer chegar, ela pode se programar melhor para ter certeza que a infraestrutura tecnológica estará preparada para atendê-los quando esse momento chegar.

[3] A infra precisa envolver o devel em suas reuniões técnicas para que o devel entenda e tenha ciência da realidade da infra, assim eles vão conseguir enxergar suas qualidades, atribuições, planos de melhorias, atualizações programadas, agendas de manutenção, eles vão conhecer os recursos disponíveis e também descobrir a limitações da equipe, sejam elas técnicas, sejam materiais. Além disto, o devel pode ser um grande aliado da infra na solução de problemas, afinal o conhecimento que o devel traz pode ajudá-los a melhorar a forma com que administram seu ambiente, tornando este processo mais eficiente.

[4] O devel precisa adotar alguma metodologia de entrega ou desenvolvimento contínuo e a infra precisa entender esse processo para que juntos criem os ambientes com as ferramentas certas.

[5] A infra precisa ceder um pouco e evoluir, precisa oferecer ao devel um ambiente adequado onde eles sejam o dono do produto, onde o devel consiga fazer todo o ciclo de desenvolvimento de forma direta, o devel precisa conseguir gerar e controlar o código, precisa fazer o commit com segurança, precisa fazer o build, testar o build, validar a aplicação e entregar a nova versão de forma transparente sem que para isso precise passar por um burocrático e engessado processo de mudança.

#### 6.1.3 Em relação aos valores humanos

Para a adoção da cultura funcionar, a equipe precisa observar e exercitar os seguintes valores:

* Confiança no trabalho de sua equipe
* Respeito pessoal e profissional por todos da equipe
* Sinceridade sobre eventos e incidentes ocorridos
* Honestidade sobre as causas dos incidentes (não esconda nada da sua equipe)
* Entendimento de que o problema é responsabilidade de todos
* Entendimento de a solução é responsabilidade de todos
* Entendimento de que os resultados são o reflexo do trabalho de toda a equipe
* Comunicação efetiva e dinâmica
* Postura construtiva sempre
* Espírito de colaboração

#### 6.1.4 Em relação a forma de trabalho

É recomendavél que a equipe:

* Internalize e adapte métodos ágeis como KABAN e SCRUM para seu dia-a-dia
* Aprofunde estudos em entrega contínua
* Aprofunde estudos em gerência de configurações e orquestração

Acho que é isso, características técnicas, valores humanos e forma de trabalhar, espero que tenha ficado claro para você.

### 6.2 Aplicando a cultura

Após observar as principais características deste movimento, normalmente pensamos em como aplicar isto em nosso meio. Para
ajudar na reflexão vamos avaliar o meio startup e o meio corporativo.

#### 6.2.1 A realidade no ambiente startup

A cultura DevOPs combina muito com startups, nestes locais normalmente já se trabalha desenvolvimento utilizando metologias
ágeis, foi inclusive neste nicho em que começaram a discutir infraestrutura ágil - a precursora do movimento devops, 
portanto, as pessoas deste meio conseguem absorver os conceitos e a cultura DevOPs sem grandes dificuldades, 
eles conseguem compreender os preceitos de colaboração e feedback pois já fazem isto em seu dia-a-dia. 

Quem está uma startup não tem as amarras e vícios da coporação, este é um grande facilitador e não é necessário
nenhum tipo de intervenção para internalizar a cultura, a partir do estímulo de um líder as pessoas começarão 
a estudar e aplicar DevOPs naturalmente.

Na startup normalmente não existe divisões, departamentos, todos trabalham juntos e isso também é um facilitador, afinal
não existem barreiras para se comunicar.

#### 6.2.2 A realidade no ambiente corporativo

A corporação não funciona como a startup, lá existe burocracia e o uso vicioso de métodos ultrapassados, portanto
não bastará o estímulo da alta hieraquia para que equipes de infra e devel comecem a vivenciar a cultura DevOPs, neste tipo
de ambiente, em minha opinião pessoal e profissional, é necessário intervir cirurgicamente para conseguir internalizar
a cultura DevOps.

Em resumo, você precisa trazer alguém - de fora - que conhece DevOP para que esta pessoa passe a contaminar os demais.

Esse processo é lento, mas se o especialista tiver os meios e o apoio do alto escalão, mudanças fantásticas poderão ocorrer.

### 6.3 O especialista DevOps no meio corporativo

Ele foi trazido para atuar como um agente de mudanças, ele precisa contaminar as áreas e mostrar que a cultura DevOps funciona.

#### 6.3.1 Característica gerais de um especialista DevOps em 2013

* Está na casa dos 30 anos ou mais
* É um profissional sênior em infraestrutura
* Tem um bom background em desenvolvimento
* Tem um bom background em metodologias ágeis
* Tem sólidos conhecimentos em soluções opensource e similares
* Trabalha intensamente com automação e infraestrutura como código

#### 6.3.2 Onde esse especialista atua?

Este especialista em DevOps terá um pé na infra e outro no devel, em alguns casos também terá o pé na área de garantia de qualidade (QA).

{% img /images/posts/devops.png [DevOps] %}

Ele é a cola que faltava para unir infra, devel e qualidade.

#### 6.3.3 Como esse especialista atua?

Ele será a ponte entre as áreas de infra e devel, ele conhece a infra a fundo e entede de forma ampla processos de desenvolvimento ágil.

##### 6.3.3.1 Pé no devel

Ele participa dos projetos de desenvolvimento desde o seu nascimento, seu foco é oferecer os recursos para os
desenvolvedores trabalhem da forma mais eficiente, além disto, com sua ótica de infra ele toma todas as precauções para que os aspectos de segurança, monitoramento, eficiência e escalabilidade sejam observados desde o início do projeto.

O DevOps vai ainda estudar todo o processo de desenvolvimento e definir - em conjunto com o devel - as ferramentas que irão permitir um processo de desenvolvimento e entrega contínua. Após definir ele vai instalar e manter esse infra.

Alguns DevOps conseguem até avaliar o código do produto e enxergar problemas de performance, esse tipo de visão sistêmica e raciocínio rápido são diferencias importantes para uma entrega com mais qualidade.

##### 6.3.3.1 Pé na infra

Na infra ele é o principal agente de mudanças, é ele que vai puxar a fila para iniciar a implantação de uma
infraestrutura ágil, ele domina as ferramentas de orquestração, gerência de configuração e provisionamento
e vai usar esse conhecimento para que a equipe passe a trabalhar a infraestrutura como código.

Este profissional também vai ajudá-los a mudar seu comportamento e cultura, ele vai orientá-los nos métodos
ágeis de execução de atividades, aqueles inspirados no SCRUM e KANBAN.

### 6.4 Ele fica na infra ou no devel?

Para internalizar DevOPs, normalmente estas barreiras não existem, infra e devel devem habitar o mesmo espaço, sem
paredes, sem divisórias, todos na mesma sala, isso é fundamental para extinguir os nichos e criar uma equipe mais unida e coesa.

Respondendo a pergunta, ele deve ficar junto com as duas equipes se for possível, esse é o melhor dos mundos.

Se não for possível ficarem todos juntos, o especialista deve se esforçar para interagir com as duas equipes diariamente.

Ele vai ser o agente de mudanças até que infra e devel comecem a entender e adotar a cultura de forma natural.

## 7. Quais os ganhos em adotar a cultura DevOPs?

Vamos avaliar dentro da ótica de cada área o que melhora com a adoção da cultura DevOPs.

### 7.1 Ganhos para a infra

* Infraestrutua como código (equipe para de administrar e passa a desenvolver a infra)
* Infra mais eficiente e rápida usando métodos ágeis
* Equipe de Infra mais organizada
* Equipe de Infra se comunicando melhor
* Infra fazendo mais em menos tempo com menos gente
* Ambientes de  gerência de configuração, orquestração e provisionamento implantados
* Deploys de infra (novos ambientes) mais rápidos e seguros => entrega rápida
* Ambiente padronizado e sob-controle
* Feedback rápido em todas as atividades de infra

### 7.2 Ganhos para o devel

* Devel tem ambiente mais adequado para trabalhar (dev/teste/prod)
* Devel passa a contar com ambiente de desenvolvimento contínuo
* Devel passa a contar com testes automatizados
* Deploys de apps (novas versões) mais rápidos e seguros => entrega rápida
* Feedback rápido em todas as fases de desenvolvimento

### 7.3 Ganhos mútuos Infra/Devel

* Acaba a divisão Infra vs Devel (acaba a guerra)
* Infra participa dos projetos e acompanha de perto tudo o que acontece
* Infra participando resulta em melhor planejamento do ambiente de produção
* Infra participando resulta em monitoramento mais eficaz da aplicação
* Devel começa a compreender melhor a infra e isso resulta em um produto melhor
* Equipes trabalhando em conjunto para aumentar o valor do negócio

### 7.4 Ganhos para a empresa

* Melhor comunicação entre devel e infra (diminuição de conflitos)
* Soluções rodando com maior estabilidade e desempenho
* Entregas mais rápidas
* Menor tempo de paradas
* Diminuição de incidentes
* Diminuição de custos
* Diminuição de riscos
* Aumento do valor do negócio

## 8. Amarrando as pontas

Vamos partir para perguntas e respostas no melhor estilo FAQ.

### 8.1 Indo direto ao ponto (e respondendo as perguntas do início)

Respondendo sobre o termo DevOps:

1. DevOps está diretamente relacionado a um melhor feedback entre as áreas de TI.

2. DevOps é um movimento, é um conceito, é uma cultura e uma filosofia, e não existe uma explicação fácil.

3. DevOps possibilita diminuição dos riscos de mudanças através do uso de um ferramental adequado e adoção de uma cultura específica.

4. DevOps busca entregar sistemas melhores, com menor custo, fazendo isto de forma mais rápida e com menor risco.

5. DevOps envolve a área de Infra e Devel primáriamente.

6. DevOps é pura metodologia ágil tando na Infra quanto no Devel.

7. DevOps só funciona se as equipes de infra e devel estiverem dispostas a ceder, mudar sua cultura e método de trabalho.

8. DevOPs não é um cargo, tão pouco um setor ou departamento, é uma cultura.

9. DevOps não está restrito ao ambiente das startups, é possível utilizar essa cultura no meio corporativo.

10. DevOps não é algo novo, as boas práticas estão ai desde de sempre, logo esse 'juntado' de práticas não é novidade para muita gente, mas para alguns serve como uma boa referência para aplicar mudanças necessárias.

Respondendo sobre o especialista em DevOps:

1. O especialista em DevOps de hoje é normalmente alguém que conhece muito de infra e tem uma base sólida de devel.

2. O especialista em DevOps também pode ser alguém que veio do devel e que tem uma base sólida de infra (esse geralmente é mais completo).

### 8.2 Quero ser um DevOps e não sei por onde começar

Não há um tutorial para isto, minha recomendação é que você estude desenvolvimento ágil e procure entender o processo
de desenvolvimento do local onde você trabalha, estude ferramentas para desenvolvimento contínuo, estude ferramentas
de gerência de configuração, orquestração e provisionamento, fazendo isto você poderá dar os primeiros passos como
entusiasta da cultura DevOps e estará apto a atuar em seu ambiente, você poderá agir como uma 
ponte entre as áreas de devel e infra, e principalmente, poderá modernizar os processos e o ambiente de infra.

### 8.3 Eu trabalho com infra ágil, sou um DevOps?

Não existe uma entidade certificadora, uma prova ou alguém que possa lhe conceder este título,
não existe um manual de conduta para dizer se você é um DevOps ou não, mas em 2013 é bastante comum encontrar 
profissionais que estão estudando infraestruta ágil e se denominando DevOps.

Se você está buscando melhorar seu ambiente, entregar mais rápido, entregar algo com mais qualidade,
algo que minimize custos, algo que diminua riscos, algo que envolva automatização pesada, se você trabalha infraestrutura como código, se 
você está atuando como um agente de mudanças em sua equipe, penso que DevOps é um termo adequado para descrever o seu trabalho, 
mesmo que não esteja diretamente envolvido com uma equipe de Devel.

### 8.4 Não existe receita mágica ou caminho rápido

Como eu já mencionei não existe uma metodologia clara, DevOps ainda é um movimento em constante construção e definição, eu citei uma série de melhorias que fazem parte da cultura DevOps, cabe a cada empresa ou a cada gestor estudar e descobrir a melhor forma de combinar essas pequenas receitas técnicas e aplicar em seu ambiente.

Espero que este artigo tenha te ajudado a entender melhor a cultura DevOPs.

## 9. Referências

Sites

* http://www.kartar.net/2010/02/what-devops-means-to-me/
* http://itrevolution.com/the-convergence-of-devops/
* http://devopsdays.org/events/
* http://devopsweekly.com

Slides

* http://www.slideshare.net/jallspaw/10-deploys-per-day-dev-and-ops-cooperation-at-flickr
* http://www.slideshare.net/KrisBuytaert/devops-the-future-is-here-its-just-not-evenly-distributed-yet
* http://www.slideshare.net/jedi4ever/devops-is-a-verb-its-all-about-feedback-13174519
* http://www.slideshare.net/jedi4ever/devops-the-war-is-over-if-you-want-it
* http://www.slideshare.net/jedi4ever/devops-tools-fools-and-other-smart-things
* http://lanyrd.com/2012/berlindevops-july/swzgt/#link-kryx
* http://www.slideshare.net/wickett/the-devops-way-of-delivering-results-in-the-enterprise

Fonte das imagens

[1] Dev OPs: https://devcentral.f5.com/blogs/us/devops-is-not-all-about-automation<br
[2] Rugby: http://imgfave.com  <br>
[3] Ops Burn: http://devops.com<br>
[4] Combusível: www.huffingtonpost.com<br>
[5] DevOps Diagram: http://www.skybotsoftware.com/skynews/2012/07/24/why-devops-more-trend<br>
