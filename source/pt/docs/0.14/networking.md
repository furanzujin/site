title: Comunicação de rede
---
Para se comunicar com outros nós (ServiceBrokers) você precisa configurar um módulo de transporte. A maioria dos módulos de transporte suportados conecta-se a um broker central de mensagens que fornece uma forma confiável de troca de mensagens entre nós remotos. Esses brokers de mensagens suportam principalmente o padrão de publicação/assinatura.

<div align="center">
    <img src="assets/networking.svg" alt="Networking diagram" />
</div>

## Transporte
Transporter é um módulo importante se você estiver executando serviços em vários nós. O transporter se comunica com outros nós. Ele transfere eventos, chama requisições e processa respostas ... etc. Se várias instâncias de um serviço estiverem sendo executadas em diferentes nós, então as solicitações serão balanceadas entre elas.

Toda a lógica da comunicação está fora da classe do módulo de transporte. Significa que é possível alternar entre módulos de transporte sem alterar linhas de código.

Existem vários módulos de transporte integrados no framework Moleculer.

### TCP transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Este é um módulo de transporte via TCP sem dependências e configuração zero. Ele usa o [Protocolo Gossip](https://en.wikipedia.org/wiki/Gossip_protocol) para disseminar status do nó, lista de serviço e sinais de vida. Contém um recurso integrado de descoberta UDP para detectar nós novos e nós desconectados na rede. Se o UDP for proibido na sua rede, use a opção `urls`. É uma lista de endpoints remotos (host/ip, port, nodeID). Pode ser uma lista estática na sua configuração ou um caminho de arquivo que contenha a lista.

**Use TCP transporter com opções padrão**
```js
// moleculer.config.js
module.exports = {
    transporter: "TCP"
};
```

**Todas as opções do TCP transporter com valores padrão**
```js
// moleculer.config.js
module.exports = {
    logger: true,
    transporter: {
        type: "TCP",
        options: {
            // Enable UDP discovery
            udpDiscovery: true,
            // Reusing UDP server socket
            udpReuseAddr: true,

            // UDP port
            udpPort: 4445,
            // UDP bind address (if null, bind on all interfaces)
            udpBindAddress: null,
            // UDP sending period (seconds)
            udpPeriod: 30,

            // Multicast address.
            udpMulticast: "239.0.0.0",
            // Multicast TTL setting
            udpMulticastTTL: 1,

            // Send broadcast (Boolean, String, Array<String>)
            udpBroadcast: false,

            // TCP server port. Null or 0 means random port
            port: null,
            // Static remote nodes address list (when UDP discovery is not available)
            urls: null,
            // Use hostname as preffered connection address
            useHostname: true,

            // Gossip sending period in seconds
            gossipPeriod: 2,
            // Maximum enabled outgoing connections. If reach, close the old connections
            maxConnections: 32,
            // Maximum TCP packet size
            maxPacketSize: 1 * 1024 * 1024            
        }
    }
};
```

**TCP transporter com lista de endpoint estática**
```js
// moleculer.config.js
module.exports = {
    nodeID: "node-1",
    logger: true,
    transporter: {
        type: "TCP",
        options: {
            udpDiscovery: false,
            urls: [
                "172.17.0.1:6000/node-1",
                "172.17.0.2:6000/node-2",
                "172.17.0.3:6000/node-3"                
            ],
        }
    }
};
```
_Você não precisa definir o parâmetro `port` porque ele encontra & analisa a porta TCP na lista de URL._

**TCP Transporter com lista de endpoint estática e abreviada** Precisa começar com `tcp://`.
```js
// moleculer.config.js
module.exports = {
    nodeID: "node-1",
    transporter: "tcp://172.17.0.1:6000/node-1,172.17.0.2:6000/node-2,172.17.0.3:6000/node-3"
};
```

**TCP Transporter com arquivo de lista de endpoint estática**
```js
// moleculer.config.js
module.exports = {
    nodeID: "node-1",
    transporter: "file://./nodes.json"
};
```

```js
// nodes.json
[
    "127.0.0.1:6001/client-1",
    "127.0.0.1:7001/server-1",
    "127.0.0.1:7002/server-2"
]
```

{% note info Serviceless node %}
Por favor, note que você não precisa listar todos os nós remotos. É suficiente que pelo menos um nó esteja online. Por exemplo, crie um nó "sem serviço", que não faz nada, apenas compartilha outros nós remotos através de mensagens redirecionadas. Portanto, todos os nós devem saber apenas o endereço do nó redirecionador para serem capazes de se comunicar com todos os outros nós.
{% endnote %}

### NATS transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Módulo de transporte integrado para [NATS](http://nats.io/).
> O Servidor NATS é um sistema de mensagens de código aberto simples, de alto desempenho, para aplicativos nativos em nuvem, mensagens IoT e arquiteturas de microsserviços.

```js
// moleculer.config.js
module.exports = {
    nodeID: "server-1",
    transporter: "nats://nats.server:4222"
};
```

{% note info Dependencies %}
Para usar este transporter, instale o módulo `nats` com o comando `npm install nats@1.4.12 --save`.
{% endnote %}

#### Exemplos
**Conecte em 'nats://localhost:4222'**
```js
// moleculer.config.js
module.exports = {
    transporter: "NATS"
};
```

**Conectar a um servidor NATS remoto**
```js
// moleculer.config.js
module.exports = {
    transporter: "nats://nats-server:4222"
};
```

**Conectar com opções**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "NATS",
        options: {
            url: "nats://localhost:4222",
            user: "admin",
            pass: "1234"
        }
    }
};
```

**Conectar com TLS**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "NATS",
        options: {
            url: "nats://localhost:4222"
            // More info: https://github.com/nats-io/node-nats#tls
            tls: {
                key: fs.readFileSync('./client-key.pem'),
                cert: fs.readFileSync('./client-cert.pem'),
                ca: [ fs.readFileSync('./ca.pem') ]
            }
        }
    }
};
```

### Redis Transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Transporte integrado para [Redis](http://redis.io/).

```js
// moleculer.config.js
module.exports = {
    nodeID: "server-1",
    transporter: "redis://redis.server:6379"
};
```
{% note info Dependencies %}
Para usar este transporter, instale o módulo `ioredis` usando o comando `npm install ioredis --save`.
{% endnote %}

#### Exemplos
**Conecte com as configurações padrão**
```js
// moleculer.config.js
module.exports = {
    transporter: "Redis"
};
```

**Conectar-se com string de conexão**
```js
// moleculer.config.js
module.exports = {
    transporter: "redis://localhost:6379"
};
```

**Conectar a um servidor Redis seguro**
```js
// moleculer.config.js
module.exports = {
    transporter: "rediss://localhost:6379"
};
```

**Conectar com opções**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "Redis",
        options: {
            host: "redis-server",
            db: 0
        }
    }
};
```

**Conectar ao cluster do Redis**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "Redis",
        options: {
            cluster: {
                nodes: [
                    { host: "localhost", port: 6379 },
                    { host: "localhost", port: 6378 }
                ]
            }
        }
    }
};
```

### MQTT Transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Módulo de transporte integrado para protocolo [MQTT](http://mqtt.org/) *(ex.: [Mosquitto](https://mosquitto.org/))*.

```js
// moleculer.config.js
module.exports = {
    nodeID: "server-1",
    transporter: "mqtt://mqtt-server:1883"
};
```
{% note info Dependencies %}
Para usar este transporter, instale o módulo `mqtt` com o comando `npm install mqtt --save`.
{% endnote %}

#### Exemplos
**Conectar com as configurações padrão**
```js
// moleculer.config.js
module.exports = {
    transporter: "MQTT"
};
```

**Conectar-se com string de conexão**
```js
// moleculer.config.js
module.exports = {
    transporter: "mqtt://mqtt-server:1883"
};
```

**Conectar ao servidor MQTT seguro**
```js
// moleculer.config.js
module.exports = {
    transporter: "mqtts://mqtt-server:1883"
};
```

**Conectar com opções**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "MQTT",
        options: {
            host: "mqtt-server",
            port: 1883,
            qos: 0,
            topicSeparator: "."
        }
    }
};
```

### AMQP (0.9) Transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Módulo de transporte integrado para o protocolo [AMQP 0.9](https://www.amqp.org/) *(ex.: [RabbitMQ](https://www.rabbitmq.com/))*.

```js
// moleculer.config.js
module.exports = {
    nodeID: "server-1",
    transporter: "amqp://rabbitmq-server:5672"
};
```
{% note info Dependencies %}
Para usar este transporter, instale o módulo `amqplib` com o comando `npm install amqplib --save`.
{% endnote %}

#### Opções do transporter
As opções podem ser passadas para o método `amqp.connect()`.

**Conectar a 'amqp://guest:guest@localhost:5672'**
```js
// moleculer.config.js
module.exports = {
    transporter: "AMQP"
});
```

**Conectar a um servidor remoto**
```js
// moleculer.config.js
module.exports = {
    transporter: "amqp://rabbitmq-server:5672"
});
```

**Conecte-se a um servidor seguro**
```js
// moleculer.config.js
module.exports = {
    transporter: "amqps://rabbitmq-server:5672"
});
```

**Conecte-se a um servidor remoto com opções & credenciais**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "AMQP",
        options: {
            url: "amqp://user:pass@rabbitmq-server:5672",
            eventTimeToLive: 5000,
            prefetch: 1,
            socketOptions: {
                servername: process.env.RABBIT_SERVER_NAME
            }
            // If true, queues will be autodeleted once service is stopped, i.e., queue listener is removed
            autoDeleteQueues: true
        }
    }
};
```

### AMQP 1.0 Transporter
![Módulo de Transporte experimental](https://img.shields.io/badge/status-experimental-orange.svg) Módulo de transporte integrado para o protocolo [AMQP 1.0](https://www.amqp.org/resources/specifications) *(ex.: [ActiveMq](https://activemq.apache.org/) ou [RabbitMQ](https://www.rabbitmq.com/) + [rabbitmq-amqp1.0 plugin](https://github.com/rabbitmq/rabbitmq-amqp1.0))*.
> Por favor note que é um transporter **experimental**. **Não use ainda em produção!**

```js
// moleculer.config.js
module.exports = {
    transporter: "amqp10://activemq-server:5672"
};
```
{% note info Dependencies %}
Para usar este transporter, instale o módulo `rhea-promise` usando o comando `npm install rhea-promise --save`.
{% endnote %}

#### Opções do transporter
As opções podem ser passadas para o método `rhea.connection.open()`, para os tópicos, as filas e as próprias mensagens.

**Conectar a 'amqp10://guest:guest@localhost:5672'**
```js
// moleculer.config.js
module.exports = {
    transporter: "AMQP10"
};
```

**Conectar a um servidor remoto**
```js
// moleculer.config.js
module.exports = {
    transporter: "amqp10://activemq-server:5672"
};
```

**Conecte-se a um servidor remoto com opções & credenciais**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        url: "amqp10://user:pass@activemq-server:5672",
        eventTimeToLive: 5000,
        heartbeatTimeToLive: 5000,
        connectionOptions: { // rhea connection options https://github.com/amqp/rhea#connectoptions, example:
            ca: "", // (if using tls)
            servername: "", // (if using tls)
            key: "", // (if using tls with client auth)
            cert: "" // (if using tls with client auth)
        },
        queueOptions: {}, // rhea queue options https://github.com/amqp/rhea#open_receiveraddressoptions
        topicOptions: {}, // rhea queue options https://github.com/amqp/rhea#open_receiveraddressoptions
        messageOptions: {}, // rhea message specific options https://github.com/amqp/rhea#message
        topicPrefix: "topic://", // RabbitMq uses '/topic/' instead, 'topic://' is more common
        prefetch: 1
    }
};
```

### Kafka Transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Módulo de transporte integrado para [Kafka](https://kafka.apache.org/).
> Trata-se de uma implementação simples. Transfere pacotes Moleculer para consumidores através de pub/sub. Não há offset implementado, recursos de replay... etc.

{% note info Dependencies %}
Para usar este transporter, instale o módulo `kafka-node` com o comando `npm install kafka-node --save`.
{% endnote %}

**Conectar ao Zookeeper**
```js
// moleculer.config.js
module.exports = {
    transporter: "kafka://192.168.51.29:2181"
};
```

**Conectar ao Zookeeper com opções personalizadas**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "kafka",
        options: {
            host: "192.168.51.29:2181",

            // KafkaClient options. Mais informações: https://github.com/SOHU-Co/kafka-node#clientconnectionstring-clientid-zkoptions-noackbatchoptions-ssloptions
            client: {
                zkOptions: undefined,
                noAckBatchOptions: undefined,
                sslOptions: undefined
            },

            // KafkaProducer options. Mais informações: https://github.com/SOHU-Co/kafka-node#producerclient-options-custompartitioner
            producer: {},
            customPartitioner: undefined,

            // ConsumerGroup options. Mais informações: https://github.com/SOHU-Co/kafka-node#consumergroupoptions-topics
            consumer: {
            },

            // Advanced options for `send`. Mais informações: https://github.com/SOHU-Co/kafka-node#sendpayloads-cb
            publish: {
                partition: 0,
                attributes: 0
            }               
        }
    }
};
```

### NATS Streaming (STAN) Transporter
![Stable transporter](https://img.shields.io/badge/status-stable-green.svg) Módulo de Transporte integrado para [NATS Streaming](https://nats.io/documentation/streaming/nats-streaming-intro/).
> Trata-se de uma implementação simples. Transfere pacotes Moleculer para consumidores através do pub/sub. Não há offset implementado, recursos de replay... etc.


```js
// moleculer.config.js
module.exports = {
    nodeID: "server-1",
    transporter: "stan://nats-streaming-server:4222"
};
```

{% note info Dependencies %}
Para usar este transporter instale o módulo `node-nats-streaming` com o comando `npm install node-nats-streaming --save`.
{% endnote %}

#### Exemplos
**Conectar com as configurações padrão**
```js
// moleculer.config.js
module.exports = {
    transporter: "STAN"
};
```

**Conectar-se com string de conexão**
```js
// moleculer.config.js
module.exports = {
    transporter: "stan://nats-streaming-server:4222"
};
```

**Conectar com opções**
```js
// moleculer.config.js
module.exports = {
    transporter: {
        type: "STAN",
        options: {
            url: "stan://127.0.0.1:4222",
            clusterID: "my-cluster"
        }
    }
};
```

### Módulo de Transporte personalizado
Um módulo personalizado de transporte pode ser criado. Recomendamos copiar a fonte do [NatsTransporter](https://github.com/moleculerjs/moleculer/blob/master/src/transporters/nats.js) e implementar os métodos de `connect`, `disconnect`, `subscribe` e ` send`.

#### Criar transporter personalizado
```js
const BaseTransporter = require("moleculer").Transporters.Base;

class MyTransporter extends BaseTransporter {
    connect() { /*...*/ }
    disconnect() { /*...*/ }
    subscribe() { /*...*/ }
    send() { /*...*/ }
}
```

#### Usar transporter personalizado

```js
// moleculer.config.js
const MyTransporter = require("./my-transporter");

module.exports = {
    transporter: new MyTransporter()
};
```

## Balanceador desativado
Alguns servidores de módulos de transporte possuem uma solução de balanceamento integrada. Ex.: RabbitMQ, NATS, NATS-Streaming. Se você quiser usar o balanceador do transporter em vez do balanceador do Moleculer, defina a opção `disableBalancer` do broker como `true`.

**Exemplo**
```js
// moleculer.config.js
module.exports = {
    disableBalancer: true,
    transporter: "nats://some-server:4222"
};
```

{% note info Please note %}
Se você desativar o balanceamento interno do Moleculer, todas as requisições & eventos serão transferidos via transporter (incluindo requisições locais). Por exemplo, você possui um serviço de cálculo matemático local e você chama `math.add` localmente, a requisição será enviada pelo transporter.
{% endnote %}

## Serialização
O transporter precisa de um módulo serializador que serializa & desserializa os pacotes transferidos. O serializador padrão é o `JSONSerializer` mas há vários serializadores integrados.

{% note warn %}
Note que certos tipos de dados (por exemplo, Date, Map, BigInt) não podem ser serializados com o serializador de JSON nativo. Se você está trabalhando com esse tipo de dados considere o uso do [Avro](#Avro-serializer), [Notepack](#Notepack-serializer) ou qualquer outro serializador binário.
{% endnote %}


**Exemplo**
```js
// moleculer.config.js
module.exports = {
    nodeID: "server-1",
    transporter: "NATS",
    serializer: "ProtoBuf"
};
```

### Serializador JSON
Este é o serializador padrão. Ele serializa os pacotes para a string JSON e desserializa os dados recebidos para o pacote.

```js
// moleculer.config.js
module.exports = {
    serializer: "JSON" // not necessary to set, because it is the default
};
```

### Serializador Avro
Serializador [Avro](https://github.com/mtth/avsc) integrado.

```js
// moleculer.config.js
module.exports = {
    serializer: "Avro"
};
```
{% note info Dependencies %}
Para usar este serializador instale o módulo `avsc` com o comando `npm install avsc --save`.
{% endnote %}

### Serializador MsgPack
Serializador [MsgPack](https://github.com/mcollina/msgpack5) integrado.

```js
// moleculer.config.js
module.exports = {
    serializer: "MsgPack"
};
```
{% note info Dependencies %}
Para usar esse serializador instale o módulo `msgpack5` com o comando `npm install msgpack5 --save`.
{% endnote %}

### Serializador Notepack
Serializador [Notepack](https://github.com/darrachequesne/notepack) integrado.

```js
// moleculer.config.js
module.exports = {
    serializer: "Notepack"
};
```
{% note info Dependencies %}
Para usar esse serializador instale o módulo `notepack` usando o comando `npm install notepack.io --save`.
{% endnote %}

### Serializador ProtoBuf
Serializador [Protocol Buffer](https://developers.google.com/protocol-buffers/) integrado.

```js
// moleculer.config.js
module.exports = {
    serializer: "ProtoBuf"
};
```
{% note info Dependencies %}
Para usar este serializador instale o módulo `protobufjs` com o comando `npm install protobufjs --save`.
{% endnote %}

### Serializador Thrift
Serializador [Thrift](http://thrift.apache.org/) integrado.

```js
// moleculer.config.js
module.exports = {
    serializer: "Thrift"
};
```
{% note info Dependencies %}
Para usar esse serializador instale o módulo `thrift` usando o comando `npm install thrift --save`.
{% endnote %}

### Serializador CBOR
CBOR ([cbor-x](https://github.com/kriszyp/cbor-x)) é [mais rápido](https://github.com/moleculerjs/moleculer/pull/905) que qualquer outro serializador.

Exemplo
```js
// moleculer.config.js
module.exports = {
    logger: true,
    serializer: "CBOR"
};
```

### Serializador personalizado
Um módulo serializador personalizado pode ser criado. Recomendamos copiar a fonte do [JSONSerializer](https://github.com/moleculerjs/moleculer/blob/master/src/serializers/json.js) e implementar os métodos `serialize` e `deserialize`.

#### Criar serializador personalizado
```js
const BaseSerializer = require("moleculer").Serializers.Base;

class MySerializer extends BaseSerializer {
    serialize(obj, type) { /*...*/ }
    deserialize(buf, type) { /*...*/ }
}
```

#### Usar serializador personalizado

```js
// moleculer.config.js
const MySerializer = require("./my-serializer");

module.exports = {
    serializer: new MySerializer()
};
```
