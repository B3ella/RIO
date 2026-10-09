## RIO
Rio é o servidor central que faz o kappalite possível.

Ele é um event store tipado pensado pra event sourcing
O tipo dos eventos é salvo em modo append-only, nos permitindo checar versões e
evoluções do esquema

Rio is a typed eventstore foucused on event sourcing
### Features
Criar e atualizar esquema - (Com check de integridade)
Consumir os eventos em massa, provavelmente num modelo pull - todos os eventos que
    aconteceram até agora, a partir de um certo offset (algumas views podem se reconstruir,
    ler kappalite).
Consumir eventos em tempo real (polling ou push model, algo perto de um webhook)
### Design
Rio vai ser feito com python em cima da foundation db
Caso python não alcance uma performance necessária vamos reescrever em go ou c

O primeiro modelo vai ser um push model pub/sub
Vamos ter um endpoint onde um consumer pode se inscrever com um offset (0 por padrão) e o
    endpoint onde o tópico espera os eventos. Rio vai começar a enviar os eventos a partir
    do offset até os eventos atuais e continuar enviando eventos enquanto o consumer estiver
    de pé. Todos os eventos seram enviados independente do tipo

Para um producer produzir eventos antes precisamos de um esquema.
    No Rio o schema de um evento é chamado de um tipo. O tipo é definido em avro com o endpoint
    Create schema
        Endpoint vai receber um schema avro nomeado, checar se é valido
        e salvar ele em blob storage
    E pode ser atualizado usando o endpoint update schema (seguindo as regras de
    compatibilidade avro)

Com os tipos necessários criados os producers podem enviar seus eventos
    um evento consiste em um tipo, um payload (os dados em si), um timestamp (adicionado
    pelo broker)
    - tipos também são informações de dominio. os eventos seguiu e deixou de seguir podem ter
    o mesmo payload, mas o tipo de evento mostra que eles tem impacto muito diferente no
    estado do sistema
    Quando um evento for emitido ele vai ser checado contra o schema pra aquele tipo
        Usando a api validate do avro
### Plano de ação
Provavelmente começar pela parte que lida com tipos
Principalmente create schema e ignorar o update schema por enquanto
Até programar os producers e consumers pelo menos
### Open questions
## Kappa lite
Kappa lite will be built on top of rio

Um Repositório central de eventos tipados
Esses eventos são usados para construir diversas views da base de dados
Essas view são construídas asincronamente. Elas se comunicam com o servidor central de eventos
por HTTP.

Um deploy simples poderia usar docker compose pra rodar varios containers, cada container é
responsável por um tipo de view (graficos, planilhas, relacionais)

Esses containers são extremamente customizaveis, já que o servidor central usa http

A maior questão é se o servidor central devia usar um padrão push ou pull.
Eu por definição mais fã de um padrão push. porém ele pode ser mais difícil de implementar
Principalmente quando consideramos que as queries vão envolver muitos dados históricos
Talvez uma abordagem mista faça sentido, pull nos dados históricos, push pras atualizações
Com o modelo Push as views me parecem mais fácil de programar

Quando uma view inicia ela lê todo histórico usando um modelo pull.
quando termina de ler o histórico, ela se inscréve pro modelo push usando o último offset que
ela recebeu na inicialização

Offsets devem ser salvos nas bases de dados derivadas. Isso possibilita que os offsets sejam
commitados na mesma transação que os dados variados. Garantindo read-once semantics
### Features in kappalite
As features são as diferentes views que oferecemos por padrão.
O servidor central vai ser o RIO
#### Monitor feed
A real time page of all events being produced by the application
Isso é literal só uma leitura direta do log.
Pra implementar isso poderiamos ter um front end react, conectado com um servidor por uma
websocket. O servidor usa a socket pra mandar os eventos
Talvez faça sentido usar um padrão rest tradicional pra mandar os primeiros eventos,
mas não tenho certeza
#### Excel
Planilhas automáticas tempo real talvez customizaveis talvez só um dump
Relativamente fácil de fazer se dermos suporte pros webhooks

## SQL Source
What about the other project I dont have a name for. SQL Sorcing?

Crud commands generating a log and automatically generating a view

The commands (create, update, delete) will be saved as an append list
A (materialized?) view will be available for the current version as a table.
But you will be able to query the history for each reccord
## Tasks
- [x] reescrever esse documento
- [x] definir a nova visão do rio/kappalite incluindo onde mora a divisão entre os dois
- [x] listar features
- [x] refazer o plano de ação
- [x] conectar com FDB
- [ ] write endpoint to create schemas
- [ ] write endpoint to produce event
- [ ] write endpoint to subscribe
- [ ] allow schema evolution (nullable fields only)
