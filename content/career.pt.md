---
title: 'Experiência'
------

## [Beblue](https://www.beblue.com.br/) — DBA Sr
*MAR DE 2019 ~ , Ribeirão Preto-SP*

Startup que oferece cashback nas transações feitas pela plataforma. Minhas principais atividades são manter o ambiente de SQL que se encontra totalmente na AWS o principal banco de dados se encontra em always-on na plataforma ECS, os bancos postgresql e mysql estão no RDS e também utilizamos a plataforma do redshift. Dentre as atividades de execução de script, melhorias de consultas e permissões estou implementando monitorias através da plataforma datadog e alguns dasboards, inclusive os alertas datadog foram criados via terraform. Devido a migração dos sistemas do legado para micro serviços os dados estão sendo gradativamente migrados do sql server para o postgres. Utilizamos o Redis como cache, mas existem algumas frentes de estudo com algumas possibilidades como ignite e o cockroach. Também atuo na manutenção do cluster swarm onde ficam os demais serviços e as aplicações que sustentam a plataforma.
O primeiro projeto entregue por mim se trata de uma imagem docker que é implantada no cluster swarm através do CI configurado no bitbucket onde automatizei e orquestrei  a execução do projeto open source schemaspy que documenta e gera diagrama de todos os bancos de dados da empresa auxiliando aos novos desenvolvedores e analistas de sistemas.

## [CAMED](https://www.camed.com.br/PortalCamed/) — DBA Sr
*ABR DE 2014 - MAR DE 2019, Fortaleza-CE*

Primeiro de tudo você vai ver o periodo e imaginar que errei o perido mais não, aqui trabalhava meio periodo na mesma epoca em que trabelhei no Banco do Nordeste. A camed acaba por ser uma holding e não só um plano de saúde com alguns consultórios proprios pois ela também responsável pela [Camed Corretora](http://www.camedseguros.com.br/) e por um creche voltada aos funcionários do BNB e da propria Camed a [Creche Paulo VI](https://www.camed.com.br/PortalCamed/tag/creche-paulo-vi/).
Aqui atuava como DBA Sr dando supor a operação sql server com cerca de 15 servicores e alguns mysql e postgresql, nossos principais sistemas eram [Benner Saúde](https://www4.benner.com.br/solucoes-para-reducao-de-custos/terceirizacao-de-servicos/area-da-saude) e os ERP da Totvs [RM](https://www.totvs.com/blog/rm-totvs/) e [Protheus](https://www.totvs.com/sistema-de-gestao/?utm_campaign=totvs_conversao_sql&utm_source=ppc&utm_medium=google_search&utm_content=ad_text_linha_produto_protheus_v2&utm_term=totvs%20protheus&hsa_tgt=kwd-22906999271&hsa_src=g&hsa_acc=5745705588&hsa_cam=1596798386&hsa_grp=61487630398&hsa_mt=e&hsa_kw=totvs%20protheus&hsa_net=adwords&hsa_ver=3&hsa_ad=323306954110&gclid=EAIaIQobChMIw46EmoLZ5gIVhYORCh0ovgdDEAAYASAAEgKZSPD_BwE) Os demais sistemas eram de desenvolvimento interno para o suporte do negocio. Minhas atividades aqui eram backup/restore, tunning, migração de dados e upgrade de versões, tinhas um cenario de replicação e muita movimentação de dados. Aqui também desenvolvi algumas soluções em Java e acabei tendo minha primeira experiência com .net atuando num sistema de integração de dados entre planos de saude através dos padroes TISS e TUSS.


## [Banco do Nordeste BNB](https://www.bnb.gov.br/) — DBA Sr
*JUL DE 2009 - MAR DE 2018, Fortaleza-CE*

Banco do Nordeste é um banco estatal de fomento da região nordeste do país focado em micro emprestimos bem atuante nos 9 estados do nordeste e com mais de 300 agências. Nessa empresa fiquei alocado por três diferentes empresas na sequência [Stefanini](https://stefanini.com/pt-br), 2009-2014, [Capgemini](https://www.capgemini.com/br-pt/). 2014-2016 e [SondaIT](https://www.sonda.com/pt-br/), 2016-2018. Independente da empresa sempre ocupei a vaga de DBA SQL Server (plenio e senior na Sefanini) integrando a equipe de DBA que chegou ater 12 membros numa opreação 24x7 com plantões aos finais de semana e feriados.
Aqui costume dizer que se você é DBA SQL Server no BNB você é DBA em qualquer lugar do mundo, administrando servidores da versão 7.5 até a 2016, stand alone e cluster de até 7 nós, replicação relacional envolvendo mais de 300 servidores, tabelas de 5 bilhões de linhas, migração de dados e upgrades de versões, tunning de query, desenvolvimento SQL (consultas, procedures, trigger), particionamento de tabelas, backup restore e mais. Ao todo um parque de quase 500 servidores sql server, todas as agências tem um servidor sql local, fora os ambientes de teste, homologação e produção. Automação de manutenções, backups e restores além de diversos script pra automação de particionamento, auditoria foram desenvolvidos além é claro da replicação que tomava um bom tempo da nossa administração. Um ambiente bem desafiador e acredito ainda continua a se-lo. Aqui também usei minhas habilidades de programador e sempre que possivel criava alguma solução em Java seja para automatizar alguns passos do nosso dia-a-dia como até um sistema de controle de horas extras que chegou a ser adotado pela empresa do contrato.


## [Lanlink Informática Ltda](https://www.lanlink.com.br/) — Trainee
*JUN DE 2008 - JUL DE 2009, Fortaleza-CE*

Como estagiário fui envolvido em um projeto interno para formação de DBA SQL Server e posteriormente assumir um cliente. Nossas atividades eram ter aulas para obter a Certificação MCITP e acompanhar o senior nos projetos e atividades diárias. Nosso mentor foi [Fábio Hemilio](https://www.linkedin.com/in/fabiohemylio/).


## Projetos como Freelancer

>  Sempre atuei como DBA e os sistemas que desenvolvia sempre eram projetos pessoais ou algum automação. Como freelancer consegui de fato vivenciar um projeto de desenvolvimento além de atuar em outros papeis dentro da tecnologia. Seguem meus perfis nas plataformas que atuo [Workana](https://www.workana.com/freelancer/9d22f6aafaf0432e235e6dc48e5f611f) e [UpWork](https://www.upwork.com/freelancers/~011b93ab5cbbf7bdca?viewMode=1)


## [MYP-7](https://myp7.com.br/) — Java Dev

Aqui fui contratado como desenvolvedor Java remoto para atuar em 2 projetos para o cliente da emrpesa que setratava de um grande atacadista do norte do país. Aqui atuei num projeto de workflow de cadastro de representates, fornecedores e produtos o que parece algo simples, porém para um grande atacadista vem a ser algo crucial devido a quantidade de registroa e cadastros a serem feitos e a ideia era integrar diretamento com ERP central. Também atuei no sistema de WMS dando manutenção em varias partes e criandos partes de endereçamento dos produtos dentro do armazém.

## [FIXPAY](https://fixpay.com.br/) — Java Dev & Devops

Aqui através de indicação de amigos fui contactado por essa nova empresa de meio de pagamentos como desenvolvedor java remoto para desenvolver inicialmente um sistema de assinatura de contratos digitais que posterioemente evoluiu também para um sistema de chamados externos e internos. Tudo isso desenvolvito on-premise, posteriormente foi pedido um sistemas para consumir os arquivos de EDI das adquirentes [Adiq](https://www.adiq.com.br/) e [Rede](https://www.userede.com.br/quero-rede?gclid=EAIaIQobChMIh9aJxIbZ5gIVEoWRCh0RRQZrEAAYASAAEgJx3_D_BwE&s_cid=mpg|gog|glp|vt-rede-aquisicao-marca-responsivo-taxa-vendas|cnr|6|sec|CRM0000|-|vt&ef_id=EAIaIQobChMIh9aJxIbZ5gIVEoWRCh0RRQZrEAAYASAAEgJx3_D_BwE:G:s&s_kwcid=AL!673!3!372703000018!e!!g!!rede) no sentido de conferencia do que era transitado. Com o crescimento dos sistemas foi sugerido e acatado a migração para o ambiente de cloud AWS utilizando as soluçẽs de EC2, S3, RDS além da impementação do jenkins ligando o codigo do bitbucket diretamente aos servidores de produção.