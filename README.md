# Projeto: Maximização de Taxa de Transferência em Redes de Computadores

Projeto desenvolvido em **C** para a disciplina de Redes de Computadores, com o objetivo de implementar e avaliar a taxa de transferência de dados utilizando **sockets TCP** e múltiplos clientes simultâneos.

## 1. Objetivo

O objetivo do projeto é maximizar e analisar a taxa de transferência de dados em uma comunicação TCP, avaliando o comportamento da transmissão conforme aumenta o número de clientes conectados simultaneamente ao servidor.

Para isso, foram desenvolvidos dois programas:

- **Servidor (`servidor.c`):** recebe os dados enviados pelos clientes, contabiliza os bytes recebidos e calcula o throughput agregado.
- **Cliente (`cliente.c`):** estabelece uma conexão TCP com o servidor e transmite continuamente dados durante um intervalo de tempo determinado.

Os testes permitem avaliar o desempenho da comunicação com diferentes quantidades de clientes e tamanhos de blocos de transmissão.

## 2. Tecnologias utilizadas

- Linguagem de programação: C
- Protocolo de transporte: TCP
- API de comunicação: Sockets (Winsock)
- Concorrência: Threads nativas do Windows (Win32)
- Compilador: GCC (MinGW)
- Sistema operacional: Windows

## 3. Funcionamento do projeto

O projeto utiliza uma arquitetura cliente-servidor.

O servidor é iniciado primeiro, informando a porta de comunicação e a quantidade de clientes que deverão se conectar.

Cada cliente estabelece uma conexão TCP com o servidor e aguarda uma autorização para iniciar a transmissão. Quando todos os clientes esperados estão conectados, o servidor libera o início do teste.

Durante o experimento:

1. Cada cliente transmite continuamente blocos de dados ao servidor.
2. O servidor utiliza uma thread independente para receber os dados de cada cliente.
3. Cada thread contabiliza individualmente a quantidade de bytes recebidos.
4. Ao término do intervalo de transmissão, os clientes encerram o envio.
5. O servidor aguarda o encerramento de todas as conexões e soma os bytes recebidos.
6. Por fim, calcula e apresenta o throughput agregado da transmissão.

### 3.1. Arquitetura

```text
                 SERVIDOR TCP
                      |
              Aguarda conexões
                      |
          +-----------+-----------+
          |           |           |
       Cliente 1   Cliente 2   Cliente N
          |           |           |
       Thread 1    Thread 2    Thread N
          |           |           |
          +-----------+-----------+
                      |
              Soma dos bytes
                  recebidos
                      |
              Cálculo do
           Throughput Agregado
```

Cada cliente possui uma conexão TCP independente com o servidor. Dessa forma, é possível avaliar a transmissão simultânea de dados e o impacto do aumento do número de conexões.

## 4. Compilação

Para compilar os programas no Windows, é necessário possuir o GCC instalado e disponível no PATH do sistema.

No terminal, execute:

**Servidor:**

```powershell
gcc -O3 -Wall servidor.c -o servidor.exe -lws2_32
```

**Cliente:**

```powershell
gcc -O3 -Wall cliente.c -o cliente.exe -lws2_32
```

A biblioteca `ws2_32` é necessária para utilizar a API Winsock, responsável pela comunicação por sockets no Windows.

## 5. Execução

### 5.1. Iniciando o servidor

O servidor recebe dois parâmetros:

```powershell
.\servidor.exe [PORTA] [NUMERO_DE_CLIENTES]
```

Exemplo:

```powershell
.\servidor.exe 5000 1
```

Nesse exemplo:
- Porta: 5000.
- Número de clientes esperados: 1.

O servidor permanecerá aguardando conexões até que todos os clientes definidos sejam conectados.

### 5.2. Iniciando os clientes

Cada cliente recebe quatro parâmetros:

```powershell
.\cliente.exe [IP_SERVIDOR] [PORTA] [DURACAO] [TAMANHO_BLOCO]
```

Exemplo:

```powershell
.\cliente.exe 127.0.0.1 5000 10 65536
```

Nesse exemplo:
- IP do servidor: `127.0.0.1` (localhost).
- Porta: 5000.
- Duração: 10 segundos.
- Tamanho do bloco: 65.536 bytes (64 KiB).

**Importante:** o servidor deve ser iniciado antes dos clientes. Para testar múltiplos clientes simultaneamente, execute cada cliente em um terminal separado.

### 5.3. Testando múltiplos clientes

Para testar com dois clientes, por exemplo:

**Terminal 1 — Servidor:**

```powershell
.\servidor.exe 5000 2
```

**Terminal 2 — Cliente 1:**

```powershell
.\cliente.exe 127.0.0.1 5000 10 65536
```

**Terminal 3 — Cliente 2:**

```powershell
.\cliente.exe 127.0.0.1 5000 10 65536
```

O servidor iniciará a transmissão após receber todas as conexões esperadas.

O mesmo procedimento pode ser repetido para diferentes quantidades de clientes, de 1 a 10.

## 6. Cálculo do throughput

O throughput representa a quantidade de dados recebidos por unidade de tempo.

Neste projeto, é calculado o throughput agregado, considerando a soma dos bytes recebidos de todos os clientes durante o experimento.

A fórmula utilizada é:

\[
Throughput = \frac{BytesRecebidos \times 8}{Tempo \times 10^6}
\]

O resultado é apresentado em Mbps (megabits por segundo).

Onde:

- `BytesRecebidos`: total de bytes contabilizados pelo servidor.
- `Tempo`: duração efetiva da transmissão em segundos.
- `8`: conversão de bytes para bits.
- `10⁶`: conversão para megabits.

O cálculo considera os bytes efetivamente retornados pelas chamadas `recv()` no servidor, e não apenas os bytes que os clientes tentaram enviar.

## 7. Diferenças entre Linux e Windows

Durante o desenvolvimento, foi necessário adaptar a implementação inicialmente planejada para Linux, utilizando as bibliotecas e funções equivalentes do Windows.

As principais diferenças são:

| Linux / POSIX | Windows / Win32 |
|---|---|
| `arpa/inet.h` | `winsock2.h` |
| `sys/socket.h` | `winsock2.h` |
| `pthread_create()` | `CreateThread()` |
| `pthread_join()` | `WaitForSingleObject()` |
| `close()` | `closesocket()` |
| `recv()` | `recv()` |
| `send()` | `send()` |
| `clock_gettime()` | `QueryPerformanceCounter()` |

### 7.1. Comunicação por sockets

No Windows, a comunicação por sockets utiliza a API Winsock. Por isso, é necessário inicializar a biblioteca com `WSAStartup()` antes de realizar operações de rede.

Ao finalizar o programa, a biblioteca deve ser encerrada utilizando `WSACleanup()`.

### 7.2. Concorrência

No Linux, a criação e o gerenciamento de threads podem ser realizados por meio da biblioteca POSIX Threads (`pthread`).

No Windows, foram utilizadas as funções nativas da API Win32, como `CreateThread()` para criar threads e `WaitForSingleObject()` para aguardar sua finalização.

### 7.3. Medição de tempo

Para medir o tempo de execução, foi utilizada a função `QueryPerformanceCounter()`, que permite realizar medições de alta precisão no Windows.

## 8. Ambiente e metodologia experimental

Os experimentos são realizados utilizando o endereço `127.0.0.1`, também conhecido como localhost.

Isso permite executar o cliente e o servidor no mesmo computador, sem depender de uma conexão externa ou de outro dispositivo.

A metodologia consiste em variar:

- Quantidade de clientes simultâneos: 1 a 10.
- Duração da transmissão.
- Tamanho dos blocos enviados.

Para cada configuração, o servidor registra a quantidade total de bytes recebidos, o tempo efetivo de execução e o throughput agregado.

Os resultados podem ser utilizados para comparar o desempenho das diferentes configurações e identificar possíveis gargalos relacionados ao processamento, gerenciamento de threads e comunicação TCP.

**Observação:** como os testes são realizados em localhost, os resultados representam o desempenho da comunicação TCP pela interface de loopback do sistema operacional. Portanto, não devem ser interpretados como a capacidade máxima da placa de rede física ou da conexão com a Internet.

## 9. Resultados experimentais

Os resultados dos testes serão organizados em tabelas e gráficos, permitindo comparar o throughput agregado conforme o número de clientes simultâneos e o tamanho dos blocos de transmissão.

| Clientes | Tamanho do bloco | Tempo (s) | Bytes recebidos | Throughput (Mbps) |
|---:|---:|---:|---:|---:|
| 1 | 65.536 bytes | 10,0007 | 18.436.653.056 | 14.748,26 |
| 2 | A preencher | A preencher | A preencher | A preencher |
| 5 | A preencher | A preencher | A preencher | A preencher |
| 10 | A preencher | A preencher | A preencher | A preencher |

Os resultados serão analisados para verificar se o aumento da concorrência proporciona ganhos de desempenho ou se provoca limitações relacionadas aos recursos computacionais.

## 10. Estrutura do projeto

```text
projeto_maximiza_rede_de_computadores/
│
├── cliente.c       # Implementação do cliente TCP
├── servidor.c     # Implementação do servidor TCP
├── cliente.exe    # Executável do cliente
├── servidor.exe   # Executável do servidor
└── README.md      # Documentação do projeto
```

## 11. Considerações finais

O projeto busca avaliar experimentalmente o desempenho de transmissões TCP com múltiplos clientes simultâneos, utilizando programação em C e técnicas de concorrência.

A partir dos experimentos, pretende-se identificar configurações que proporcionem maior throughput agregado, analisar o impacto do número de conexões e dos tamanhos dos blocos e discutir os possíveis gargalos encontrados durante a execução.

---

**Disciplina:** Redes de Computadores  
**Professor:** Arlindo F. da Conceição  
**Linguagem:** C  
**Protocolo:** TCP  
**Sistema operacional:** Windows
