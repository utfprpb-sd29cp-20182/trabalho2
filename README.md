## Disciplina de Sistemas Distribuídos - Engenharia de Computação - UTFPR - 2018/2
# Trabalho 2 - Serviço de Log Distribuído

## Visão Geral
Os relógios lógicos vetoriais são usados para estabelecer a ordem parcial de eventos em um sistema distribuído, permitindo que os usuários determinem a relação de causalidade em um sistema.

Esse trabalho visa explorar esse conceito, por meio da implementação de um serviço de *Log* de eventos que acontecem em processos em um sistema distribuído com ordenamento parcial.  O serviço consiste em dois tipos de processos:
* *Log*: que registra os eventos que ocorreram nos processos *workers*;
* *Workers*: processos que interagem entre si para execução de uma aplicação distribuda qualqer.

Na figura abaixo é apresentado um exemplo de troca de mensagens entre os processos *workers* P<sub>1</sub> , P<sub>2</sub> , P<sub>3</sub> e o processo *Log*. Quando um processo worker processa um evento (i.e. evento interno, enviar ou receber mensagem), esse processo deve enviar uma mensagem para o processo *Log*. O processo *Log*consegue fazer uma ordenação FIFO das mensagens que são entregues a ele oriundas de um mesmo processo. Contudo, essa ordenação não seria suficiente em cenários em que a mensagem enviada por um processo P<sub>i</sub> para *Log* é consequência de uma mensagem que P<sub>i</sub> recebeu de P<sub>j</sub>. Por exemplo, no caso 1 da Figura 1, o processo P<sub>2</sub> envia uma mensagem para P<sub>3</sub> e na sequência envia uma mensagem para Log. P<sub>3</sub>, ao receber a mensagem de P<sub>2</sub>, também envia uma mensagem para o processo *Log*. Nesse exemplo, a ordem das mensagens entregues ao processo *Log* segue a mesma ordem de criação dessas mensagens. Contudo, no cenário 2 da Figura 1, a mensagem enviada por P<sub>2</sub> para o processo *Log* chega antes da mensagem enviada por P<sub>1</sub> para o processo *Log*, não respeitando assim a ordem dos eventos que geraram tais mensagens.

![dlog](https://github.com/utfprpb-sd29cp-20182/trabalho2/blob/master/dlog-messages.png)

Este trabalho terá duas entregas (versões 1.0 e 2.0).  


## Versão 1.0 (7 pontos) - sem ordenação das mensagens no processo *Log*
### Processo *Log*
O processo *Log* deve: 
1. Receber a lista de processos (workers) que poderão enviar mensagens para ele (essa lista deverá ser fornecida por meio de um arquivo texto, sendo que cada linha do arquivo deve representar o identificador único de um processo – por exemplo: `p1`); 
2. Receber as mensagens, indicando o evento que ocorreu juntamente com vetor de relógio lógico, dos processos workers. 
*Obs*: apesar do processo *Log* receber o vetor de relógio lógico, esse ainda não fará uso do mesmo. Isso será explorado na Versão 2.0.
3. Imprimir na tela as mensagens assim que forem recebidas.  Na tela do processo de Log deve ser impresso algo como:
```c
#Timestamp            FROM  Message   LogicalClock
2018-10-18 15:50:00   P1    m1        [1,0,0]
```
Sendo que o `timestamp` que aparece na tela seria obtido a partir do relógio fı́sico da máquina onde o
processo Log está em execução. Esse `timestamp` indica a hora exata que a linha foi impressa na
tela, isto é, não possui qualquer relação com horário que a mensagem foi recebida pelo processo
*Log*.

Esse projeto poderá ser desenvolvido em C ou em Java, usando se sockets ou RPC/RMI.

### Processo *Worker*
O processo *worker* deve:
1. Enviar uma mensagem para o processo Log a cada evento (local, envio de mensagem ou recebimento de mensagem). Juntamente com a mensagem, deve ser enviado o vetor de relógio lógico do processo.
2. Processar uma lista de eventos, fornecida por meio de um arquivo texto. Deve ser aguardado um tempo entre o processamento dos eventos.
3. Ser capaz de processar mensagens recebidas de outros processos. 
4. Para efeitos de depuração, deve ser impresso na tela cada evento processado.

Para sua execução o processo *worker* deve possuir:
* Um identificador único; 
* Uma semente única para a geração de números pseudo-aleatórios (um número inteiro)
* Um valor, em milissegundos, para o tempo que o processo ficará aguardando antes de processar o próximo evento; 
* Um valor, em milissegundos, para variação de atraso (*jitter*) máximo entre o envio de uma mensagem para um processo worker e o envio de uma mensagem para o processo *Log*. O jitter para cada mensagem deve ser obtido por meio de um número
aleatório de 0 até o valor máximo fornecido. 
* Um arquivo texto com a identificação dos demais processos (Log e demais workers). 
* Um arquivo texto com a lista de eventos a serem processados.  

Essas informações devem ser fornecidas como argumentos de linha de comando, conforme exemplo em Java:
```java
# java nomeDaClasse Id Semente tempoEspera tempoJitter arquivo-com-processos arquivo-com-
eventos
java processoWorker p1 123456 3000 2000 processos.txt eventos.txt
```
Cada linha do arquivo `processos.txt` deve identificar um processo. Desse modo, o formato desse arquivo será conforme apresentado abaixo:
```java
p1
p2
p3
Log
```
Cada linha do arquivo `eventos.txt` deve identificar o tipo do evento a ser processado, podendo ser: (l) indicando um evento local ou (e) indicando um evento de envio de uma mensagem para outro processo *worker*.  No evento de envio, a mensagem deve ser enviada para um processo destino escolhido de forma aleatória.  O conteúdo/mensagem do evento pode ser qualquer coisa, como um número inteiro, uma letra, um texto. Exemplo, com conteúdo do evento representado por uma letra:
```java
l,a
e,b
e,c
l,d
e,e
```

Um processo *worker* só pode iniciar o processamento do arquivo de eventos quando todos os demais processos já estiverem sido instanciados. Sendo assim, deve-se fazer um sincronizador para garantir isso (um laço que não permitirá a progressão até que os demais estejam instanciados). 

Abaixo o pseudo-código para o processo worker:
```java
sincronizarProcessos(); // só prosseguirá se todos os processos estiverem instanciados
while(...){
aguardarTempoAleatorio();
processarEvento(); // lido do arquivo ou recebido de outro processo
jitter();
enviarParaLog();
}
```

### Validação da Versão 1.0
Para validar a Versão 1.0 monte um cenário com 4 processos *workers* e execute alguns experimentos. Verifique se a ordem das mensagens impressas na tela do processo *Log* está de acordo com a ordem que essas mensagens foram criadas.

## Versão 2.0 (3 pontos) - com ordenação das mensagens no processo *Log*
### Processo *Log*
Se o processo *Log* imprimir na tela a mensagem assim que ele a receber (assim como na Versão 1.0), então esse poderá
imprimir em uma ordem errada. Nessa versão, o processo *Log* deve imprimir as mensagens que receber na ordem correta.

Para isso o processo *Log* tem duas opções:
1.  Receber todas as mensagens, guardá-las em uma fila e só imprimir na tela depois de todos os processos workers forem encerrados e com base nos relógios lógicos fazer o ordenamento das mensagens.
2.  Imprimir as mensagens na ordem correta e com os processos workers ainda em
execução. Para isso pode se fazer uso de uma fila de espera de mensagens no processo *Log* e este ainda deveria possuir um relógio para registrar os *timestamps* das últimas mensagens recebidas de cada processo worker. Quando uma nova mensagem chegar no processo *Log*, este deve atualizar o relógio, armazenar a mensagem na fila de espera e percorrer a fila em busca de mensagens que poderiam ser impressas.

Sugestão: primeiro implemente considerando a primeira opção.

### Validação da Versão 2.0
Para validar a Versão 2.0, execute o mesmo cenário da versão 1.0.  Verifique se a ordem das mensagens impressas na tela do processo *Log* está de acordo com a ordem que essas mensagens foram criadas.


# Avaliação e Datas de Entrega
* Entregar via Github Classroom, conforme cronograma de entregas abaixo:
  - 22 e 25/10/2018 - Commit ao final da aula com o que foi desenvolvido, deve haver evolução entre um commit e outro.
  - até 08/11/2018 às 13h50min - Commit da versão 1.0/2.0 (final) e apresentação em sala. 
  - Faça uso das  [tags](https://git-scm.com/book/pt-br/v1/Git-Essencial-Tagging) do git para marcar os commits.
  
* o trabalho poderá ser feito individualmente ou em dupla;
* dentro do projeto deve haver um arquivo leiame.txt contendo:
  – nome dos integrantes da equipe;
  – informações para compilação e execução do programa
  
* A avaliação será feita levando em consideração:
  – se atende aos requisitos solicitados;
  – estruturação e clareza do código;
  – avaliação individual de cada participante. Todos os integrantes devem saber responder a qualquer questionamento do professor sobre a implementação de tópicos do trabalho.
  
