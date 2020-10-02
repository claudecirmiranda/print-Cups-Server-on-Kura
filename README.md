<h2>Eclipse Kura - Implementando Impressão CUPS</h2>

[![Kura](http://soaone.com.br/kura/kura_logo_small.png "Kura")](http://soaone.com.br/kura/kura_logo_small.png "Kura")

O Eclipse Kura oferece uma plataforma que pode viver na fronteira entre a rede de dispositivo privada e a rede local, Internet pública ou rede celular, fornecendo um gateway gerenciável e inteligente para essa fronteira, capaz de executar aplicativos que podem coletar informações coletadas localmente e entregá-las de forma confiável para a nuvem.

Kura, por estar dentro de um contêiner OSGi - Eclipse Equinox , trabalha com uma estrutura padrão para manipulação de eventos, código de empacotamento e uma variedade de serviços padrão. Um aplicativo no Kura é entregue como um módulo OSGi e é executado dentro do contêiner junto com os outros componentes do Kura. Usando a biblioteca Eclipse Paho MQTT, Kura fornece um serviço de repositório `¹ store-and-forward` para esses aplicativos para obter as informações coletadas dos dispositivos conectados localmente ou dispositivos conectados à rede, enviando esses dados para intermediários MQTT e outros serviços em nuvem.

O pacote Kura é completado com um front end da web que permite ao desenvolvedor ou administrador fazer login remotamente e configurar todos os pacotes compatíveis com OSGi e que os desenvolvedores podem utilizar para fornecer um aspecto voltado para a web para as necessidades de configuração de seus próprios aplicativos. Os desenvolvedores devem descobrir que o conjunto de ferramentas OSGi do Eclipse IDE torna a rota da concepção do código à instalação no Kura um caminho facilmente navegável também.

Com uma combinação de um contêiner OSGi, mensagens MQTT, uma ampla variedade de rede e conectividade local, web remota e controle de linha de comando e o suporte familiar de ferramentas Eclipse, Kura está definida para fornecer uma opção atraente para desenvolvedores corporativos e IoT / M2M Java .

<h3>Sobre esse projeto</h3>

Esse projeto destina-se a construção de um componente para o Eclipse Kura, com a finalidade de imprimir no CUPS Server Printer.

[![arquiteturar](http://soaone.com.br/kura/architecture.png "Arquitetura")](http://soaone.com.br/kura/architecture.png "Arquitetura")

<h3>Preparação do Ambiente</h3>

Para prover o ambiente necessário para implementação da solução proposta, será necessário a disponibilização de 3 containers docker (Kura, Spring Boot API REST e CUPS).

[![arquiteturar](http://soaone.com.br/kura/containers.png "Arquitetura")](http://soaone.com.br/kura/containers.png "Arquitetura")

<b>Para subir os containers no docker:</b>

- **Kura**


	docker run -d -v /usr/src/app/kura/packages/:/opt/eclipse/kura/data/packages \
	-v /usr/src/app/kura/snapshots:/opt/eclipse/kura/user/snapshots \
	-p 8080:8080 -p 8090:8090 -p 9123:9123 \
	--name kura -t eclipse/kura --restart unless-stopped

- **Spring Boot REST API**


	docker run -d -it -p 8082:8082 --name apicups 12254/cupsprinter:latest

- **CUPS Server**


	docker run -d -p 631:631 -v /var/run/dbus:/var/run/dbus --name cupsd olbat/cupsd

Clonar o projeto conforme instruação abaixo:

	$ git clone https://grupomult@dev.azure.com/grupomult/VLI-Logistica/_git/vli_kura_java_impressao_servidor

Importar o projeto na IDE do Eclipse Kura e gerar o pacote `br.com.vli.kura.wire.cups.printer_1.0.200-SNAPSHOT.dp`.

**Passoa a passo para importação do pacote no Kura:**

1. Acessar o console web do Kura;
`http://localhost:8080/kura`
2.  Clicar no menu Packages
[![Packages](http://soaone.com.br/kura/package1.png "Packages")](http://soaone.com.br/kura/Package1.png "Packages")
3.  Clicar no botão [![Packages](http://soaone.com.br/kura/package2.png "Packages")](http://soaone.com.br/kura/Package2.png "Packages")
4. Será aberto uma janela popup para selecionar o pacote a ser instalado;
[![Packages](http://soaone.com.br/kura/package3.png "Packages")](http://soaone.com.br/kura/Package3.png "Packages")
5. Clique no botão `Escolher arquivo`;
6. Escolha o arquivo `br.com.vli.kura.wire.cups.printer_1.0.200-SNAPSHOT.dp`;
7. Clique no botão `Submit`;
8. O novo componente estará disponível no Wire Componentes do Wire Graph.
[![Packages](http://soaone.com.br/kura/package4.png "Packages")](http://soaone.com.br/kura/Package4.png "Packages")

**Utilizando o componente Cups Printer no Wire Graph**

1. Arraste o componente Cups Printer para o Wire Graph e dê um nome para ele e clique no botão `Ok`;
[![Component](http://soaone.com.br/kura/makewiregraph1.png "Component")](http://soaone.com.br/kura/makewiregraph1.png "Component")
2. Clique no componente recém adicionado no Wire Graph e configure os option;
[![Component](http://soaone.com.br/kura/makewiregraph2.png "Component")](http://soaone.com.br/kura/makewiregraph2.png "Component")

**IP do Hots** preencha com o IP/Hostname do CUPS Server;
**Port** preencha com a porta do CUPS Server, geralmente é a 631;
**Nome da Impressora** preencha com o nome da impressora configurada no CUPS Server;
**URI da API** aqui será informado a URI completa da API que irá imprimir o documento no CUPS Server, trocar o `localhost` do endereço default pelo IP ou Hostname do sever que está executando o container da API.

Terminado a configuração do componente, faça a ligação necessária dos outros componente nele e dele a outros componentes, no Wire Graph, conforme o exemplo abaixo:

[![Component](http://soaone.com.br/kura/makewiregraph3.png "Component")](http://soaone.com.br/kura/makewiregraph3.png "Component")

Clique no botão `Apply` e o Wire Graph estará pronto.

**Testando o componente**

Para fins de ilustração, nosso Wire Graph contém 3 componentes:

[![Component](http://soaone.com.br/kura/makewiregraph4.png "Component")](http://soaone.com.br/kura/makewiregraph4.png "Component")

Um componente Camel Consumer API-REST;
Um componente Cups Printer;
Um componente Logger.

O componente Camel Consumer API-REST será responsável por capturar os dados de uma requisição `POST`, de um serviço REST, com os seguintes parâmetros de entrada:

	{
	    "cabecalho": {
	        "idCorrelacao": "630a42b2-8ca4-4c5e-88d8-d53e10ee4097",
	        "dataHora": "2019-06-17T14:16:36.237Z"
	    },
	    "corpo": {
	        "operacao": {
	            "tags": [
	                {
	                    "id": "IMPRESSORA_02_TAG_ESCRITA_02",
	                    "valor": "Imprimir",
	                    "ticket": "identificador_ticket",
	                    "nome": "nota_fiscal_01",
	                    "tipo": "pdf",
	                    "conteudo": "dGVzdGUgZGUgaW1ucHJlc3Nhbw==",
	                    "guid" : "11"
	                }
	            ]
	        }
	    }
	}

Quando a solicitação for executada, o component Camel Consumer - API-Rest, irá capturar os dados de retorno da API em seguida, no fluxo, o componente Cups Printer, que está ligado a ele, será executado.

O componente Cups Printer, lê o conteúdo do wire record do componente Camel Consumer - API-Rest, para pegar alguns dados para serem fornecidos para  chamada da API Spring Boot, que irá imprimir de fato no CUPS Server.

A API Spring Boot, chamada pelo componente Cups Printer é executada, imprime o documento fornecido pelo wire record do componente Camel Consumer - API-Rest, e chama o componente Logger.

O componente Logger exibe os logs da execução do fluxo.

O componente Cups Printer, retorna o seguinte wire record:

	Record List content:
	 Record content:
	dados :  {
	    "id": "IMPRESSORA_02_TAG_LEITURA_02",
	    "valor": "Imprimir",
	    "erro": "null",
	    "nome": "teste de imnpressao",
	    "ticket": "idTicketImpressao",
	    "dataHora": "2020-10-02T09:52:10.836Z",
	    "guid": "388cbcd1-11e6-4ca4-82d8-7c3a92712700"
	 }
	operacao : LeituraTicket
	asset : IMPRESSORA_02

O documento teve a sua impressão realizada com sucesso, pelo fluxo do Kura.


<h2>ANEXOS<h2>
<h3>¹ Comutação de pacotes store-and-forward</h3>

[![Store-and-forward](http://soaone.com.br/kura/comutacaodepacote.jpg "Store-and-forward")](http://soaone.com.br/kura/comutacaodepacote.jpg "Store-and-forward")

Dentro do contexto em que operam os protocolos da camada rede, os principais componentes são o equipamento do provedor de telecomunicações (roteadores conectados por linhas de transmissão), apresentados na elipse sombreada da figura, e o equipamento dos clientes mostrados fora da elipse. O host H1 está diretamente conectado a um dos roteadores do provedor de telecomunicações, denominado A, por uma linha dedicada. No outro lado, o host H2 está em uma LAN com um roteador F pertencente ao cliente e operado por ele.

Um host com um pacote a enviar o transmite para o roteador mais próximo, seja em sua própria LAN ou sobre um enlace ponto a ponto para o provedor de telecomunicações. O pacote é armazenado ali até chegar totalmente, de forma que o total de verificação possa ser conferido. Em seguida, ele é encaminha para o próximo roteador ao longo do caminho, até alcançar o host de destino, onde é entregue. Esse mecanismo é a comutação de pacotes store-and-forward.


