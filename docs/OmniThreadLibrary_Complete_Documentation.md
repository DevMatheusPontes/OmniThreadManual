# OmniThreadLibrary - Documentação Completa

## Índice
1. [Introdução](#introdução)
2. [Fundamentos de Multi-threading](#fundamentos-de-multi-threading)
3. [Introdução ao OmniThreadLibrary](#introdução-ao-omnithreadlibrary)
4. [Multi-threading de Alto Nível](#multi-threading-de-alto-nível)
5. [Multi-threading de Baixo Nível](#multi-threading-de-baixo-nível)
6. [Sincronização](#sincronização)
7. [Recursos Diversos](#recursos-diversos)
8. [Guias Práticos](#guias-práticos)
9. [Apêndices](#apêndices)

---

## Introdução

### Sobre o Autor
**Primož Gabrijelčič** é um desenvolvedor experiente especializado em aplicações de servidor de alta disponibilidade. Ele é o criador de projetos open-source como GpProfile e OmniThreadLibrary, além de ser professor e autor de livros técnicos.

### Créditos
- **Gorazd Jernejc**: Contribuições significativas para o projeto
- **Pierre le Riche**: FastMM (gerenciador de memória)
- **Joe C. Hecht**: Revisor técnico
- **LeanPub**: Plataforma de publicação
- **ProWritingAid**: Ferramenta de edição

### Versão da Documentação
Esta documentação cobre a **OmniThreadLibrary versão 3.07.7**.

### Convenções de Formatação
- `[version tag]`: Indica versões específicas da biblioteca
- `[r1184]`: Referências a revisões específicas
- `[1.0-1.1]`: Intervalos de versões

### Recursos de Aprendizado
- **Demos**: Exemplos práticos incluídos
- **Webinars**: Sessões de treinamento online
- **Blog**: Artigos técnicos e atualizações
- **StackOverflow**: Comunidade de perguntas e respostas
- **Fórum OTL**: Comunidade oficial da biblioteca

---

## Fundamentos de Multi-threading

### Conceitos Básicos

#### Processos vs Threads
- **Processo**: Instância de um programa em execução com seu próprio espaço de memória
- **Thread**: Unidade de execução dentro de um processo, compartilhando memória
- **Thread Principal**: Thread que executa a interface do usuário
- **Threads de Background**: Threads que executam tarefas em paralelo

#### A Ilusão de Paralelismo
Em sistemas com múltiplos núcleos, threads podem executar verdadeiramente em paralelo. Em sistemas single-core, o sistema operacional alterna rapidamente entre threads, criando a ilusão de paralelismo.

### Problemas do Multi-threading

#### 1. Leitura e Escrita de Dados Compartilhados

**Problema**: Acesso não sincronizado a variáveis compartilhadas pode causar condições de corrida.

```delphi
var
  a, b: integer;

// Thread 1
a := 42;
// Thread 2
b := a; // Pode ler valor inconsistente
```

#### 2. Modificação de Dados Compartilhados

**Problema**: Operações de incremento/decremento não são atômicas.

```delphi
var
  data: integer;

data := 0;

// Thread 1
for i := 1 to 100000 do
  Inc(data);
// Thread 2
for i := 1 to 100000 do
  Dec(data);
// Resultado final pode não ser 0!
```

#### 3. Escritas Disfarçadas de Leituras

**Problema**: Operações que parecem ser de leitura podem modificar estado interno.

```delphi
var
  data: TStream;

// Worker thread
data.Read(...);
// Main thread
UpdatePosition(data.Position / data.Size); // Position pode ter mudado!
```

### Soluções de Sincronização

#### 1. Operações Atômicas

**TOmniAlignedInt32/TOmniAlignedInt64**: Variáveis inteiras alinhadas para operações atômicas.

```delphi
var
  data: TOmniAlignedInt32;

data := 0;

// Thread 1
for i := 1 to 100000 do
  data.Increment;
// Thread 2
for i := 1 to 100000 do
  data.Decrement;
// Resultado final será 0!
```

#### 2. Operações Interlocked

**TInterlocked**: Operações thread-safe do sistema.

```delphi
var
  data: integer;

data := 0;

// Thread 1
for i := 1 to 100000 do
  TInterlocked.Increment(data);
// Thread 2
for i := 1 to 100000 do
  TInterlocked.Decrement(data);
```

#### 3. Seções Críticas

**TOmniCS**: Seções críticas para sincronização.

```delphi
var
  data: integer;
  lock: TOmniCS;

data := 0;

// Thread 1
for i := 1 to 100000 do begin
  lock.Acquire;
  Inc(data);
  lock.Release;
end;

// Thread 2
for i := 1 to 100000 do begin
  lock.Acquire;
  Dec(data);
  lock.Release;
end;
```

#### 4. Abordagem Preferida: Cópia e Agregação

**Filosofia OTL**: Evitar bloqueios sempre que possível.

```delphi
var
  data: integer;

// Thread 1
var
  tempData: integer;
tempData := 0;
for i := 1 to 100000 do
  Inc(tempData);
send tempData to main thread

// Thread 2
var
  tempData: integer;
tempData := 0;
for i := 1 to 100000 do
  Dec(tempData);
send tempData to main thread

// Main thread
data := result from thread 1 + result from thread 2
```

---

## Introdução ao OmniThreadLibrary

### Requisitos
- **Delphi**: Versão 2009 ou superior
- **Free Pascal**: Versão 3.0 ou superior
- **Windows**: XP ou superior
- **Linux**: Suporte experimental

### Licença
OmniThreadLibrary é distribuída sob a licença **Mozilla Public License 1.1**.

### Instalação

#### 1. Instalação via GetIt
1. Abra o Delphi
2. Vá para Tools → GetIt Package Manager
3. Procure por "OmniThreadLibrary"
4. Clique em Install

#### 2. Instalação via Delphinus
1. Instale o Delphinus Package Manager
2. Procure por "OmniThreadLibrary"
3. Instale o pacote

#### 3. Instalação do Design Package
Para componentes visuais, instale o pacote de design adicional.

### Por que usar OmniThreadLibrary?

#### Vantagens
- **Abstração de Alto Nível**: Simplifica programação multi-thread
- **Padrões Modernos**: Suporte a async/await, futures, pipelines
- **Performance**: Otimizada para máxima eficiência
- **Flexibilidade**: Suporte tanto para alto quanto baixo nível
- **Comunidade**: Ativa comunidade de desenvolvedores

### Tasks vs Threads

#### Tasks (Recomendado)
- **Abstração**: Alto nível, fácil de usar
- **Gerenciamento**: Automático de ciclo de vida
- **Pooling**: Reutilização eficiente de threads
- **Comunicação**: Sistema de mensagens integrado

#### Threads Tradicionais
- **Controle**: Baixo nível, mais controle
- **Complexidade**: Maior complexidade de gerenciamento
- **Performance**: Potencialmente mais rápida para casos específicos

### Locking vs Messaging

#### Locking (Evitar)
- **Problemas**: Deadlocks, contenção, complexidade
- **Performance**: Pode degradar performance
- **Manutenção**: Código difícil de manter

#### Messaging (Preferido)
- **Segurança**: Evita condições de corrida
- **Simplicidade**: Código mais limpo e legível
- **Escalabilidade**: Melhor para sistemas complexos

### Message Loop Necessário

#### Aplicações GUI
OmniThreadLibrary requer um message loop ativo para funcionar corretamente.

#### Aplicações Console
Para aplicações console, é necessário criar um message loop manual:

```delphi
program ConsoleApp;
uses
  Windows, Messages;

var
  msg: TMsg;
begin
  while GetMessage(msg, 0, 0, 0) do begin
    TranslateMessage(msg);
    DispatchMessage(msg);
  end;
end;
```

### TOmniValue

#### Visão Geral
`TOmniValue` é uma classe wrapper que permite passar qualquer tipo de dados entre threads de forma type-safe.

#### Acesso a Dados

```delphi
var
  value: TOmniValue;
  intVal: integer;
  strVal: string;

// Atribuição
value := 42;
value := 'Hello World';

// Acesso
intVal := value.AsInteger;
strVal := value.AsString;
```

#### Teste de Tipos

```delphi
if value.IsInteger then
  ShowMessage('É um inteiro: ' + IntToStr(value.AsInteger))
else if value.IsString then
  ShowMessage('É uma string: ' + value.AsString);
```

#### Limpeza de Conteúdo

```delphi
value.Clear; // Remove o conteúdo
```

#### Operadores

```delphi
var
  v1, v2, v3: TOmniValue;

v1 := 10;
v2 := 20;
v3 := v1 + v2; // v3 = 30
```

#### Tipos Genéricos

```delphi
var
  value: TOmniValue;
  list: TList<string>;

list := TList<string>.Create;
value := list;
// TOmniValue gerencia automaticamente a referência
```

#### Acesso a Arrays

```delphi
var
  value: TOmniValue;
  arr: TArray<integer>;

arr := [1, 2, 3, 4, 5];
value := arr;

// Acesso aos elementos
for i := 0 to value.AsArrayLength - 1 do
  ShowMessage(IntToStr(value.AsArrayItem[i].AsInteger));
```

#### Manipulação de Records

```delphi
type
  TMyRecord = record
    ID: integer;
    Name: string;
  end;

var
  value: TOmniValue;
  rec: TMyRecord;

rec.ID := 1;
rec.Name := 'Test';
value := rec;
```

#### Propriedade de Objetos

```delphi
var
  value: TOmniValue;
  obj: TMyObject;

obj := TMyObject.Create;
value := obj; // TOmniValue assume propriedade
// obj será liberado automaticamente quando value for destruído
```

#### Trabalhando com TValue

```delphi
var
  value: TOmniValue;
  tval: TValue;

tval := 42;
value := tval;
```

### TOmniValueObj

Classe auxiliar para trabalhar com objetos em `TOmniValue`.

### Interfaces Fluentes

OmniThreadLibrary utiliza interfaces fluentes para criar código mais legível:

```delphi
CreateTask(MyWorker)
  .OnMessage(HandleMessage)
  .OnTerminated(HandleTerminated)
  .Run;
```

---

## Multi-threading de Alto Nível

### Introdução

#### Ciclo de Vida de uma Abstração
1. **Criação**: Definir a tarefa
2. **Configuração**: Configurar parâmetros
3. **Execução**: Iniciar a tarefa
4. **Comunicação**: Trocar mensagens
5. **Finalização**: Aguardar conclusão

#### Métodos Anônimos, Procedimentos e Métodos

```delphi
// Método anônimo
CreateTask(
  procedure
  begin
    // código da tarefa
  end
).Run;

// Procedimento
procedure MyWorker;
begin
  // código da tarefa
end;

CreateTask(MyWorker).Run;

// Método de classe
CreateTask(MyClass.MyMethod).Run;
```

#### Pooling

OmniThreadLibrary utiliza pooling automático de threads para melhor performance:

```delphi
// Pooling automático
CreateTask(MyWorker).Run; // Reutiliza thread do pool

// Pooling manual
var
  pool: IOmniThreadPool;
begin
  pool := CreateThreadPool('MyPool');
  pool.MaxExecuting := 4; // Máximo 4 threads simultâneas
end;
```

### Blocking Collection

#### IOmniBlockingCollection

Coleção thread-safe que bloqueia quando vazia ou cheia.

```delphi
var
  collection: IOmniBlockingCollection<integer>;

collection := TOmniBlockingCollection<integer>.Create;

// Producer
for i := 1 to 100 do
  collection.Add(i);
collection.CompleteAdding;

// Consumer
while not collection.IsCompleted do begin
  if collection.TryTake(item) then
    ProcessItem(item);
end;
```

#### Importação e Exportação em Massa

```delphi
// Importação em massa
var
  items: TArray<integer>;
begin
  items := [1, 2, 3, 4, 5];
  collection.AddRange(items);
end;

// Exportação em massa
var
  allItems: TArray<integer>;
begin
  allItems := collection.ToArray;
end;
```

#### Throttling

Controle de taxa de processamento:

```delphi
collection := TOmniBlockingCollection<integer>.Create;
collection.Throttle(100); // Máximo 100 itens por segundo
```

### Configuração de Tarefas

#### Configurações Básicas

```delphi
CreateTask(MyWorker)
  .Name('MyTask')                    // Nome da tarefa
  .Priority(tpNormal)                // Prioridade
  .OnMessage(HandleMessage)          // Handler de mensagens
  .OnTerminated(HandleTerminated)    // Handler de término
  .Run;
```

#### Configurações Avançadas

```delphi
CreateTask(MyWorker)
  .SetParameter('param1', 'value1')  // Parâmetros
  .SetParameter('param2', 42)
  .MonitorWith(MyMonitor)            // Monitoramento
  .NoMessageQueue                    // Sem fila de mensagens
  .Run;
```

### Async

#### Execução Assíncrona Simples

```delphi
var
  task: IOmniTask;

task := Async(
  procedure
  begin
    // Código executado em background
    Sleep(5000);
  end
);

// Código continua executando...
// task será executada em paralelo
```

#### Tratamento de Exceções

```delphi
var
  task: IOmniTask;

task := Async(
  procedure
  begin
    try
      // Código que pode gerar exceção
      RiskyOperation;
    except
      on E: Exception do
        LogError(E.Message);
    end;
  end
);
```

### Async/Await

#### Padrão Moderno de Programação Assíncrona

```delphi
procedure TMyClass.DoAsyncWork;
var
  result: string;
begin
  // Execução assíncrona
  result := await Async(
    function: string
    begin
      Sleep(2000); // Simula trabalho pesado
      Result := 'Trabalho concluído!';
    end
  );
  
  ShowMessage(result);
end;
```

### Future

#### IOmniFuture<T> Interface

Representa um valor que será calculado no futuro.

```delphi
var
  future: IOmniFuture<integer>;

future := Future<integer>(
  function: integer
  begin
    Sleep(3000); // Simula cálculo demorado
    Result := 42;
  end
);

// Código continua executando...
// Em algum momento:
ShowMessage('Resultado: ' + IntToStr(future.Value));
```

#### Detecção de Conclusão

```delphi
if future.IsCompleted then
  ShowMessage('Tarefa concluída!')
else
  ShowMessage('Tarefa ainda executando...');
```

#### Cancelamento

```delphi
future.Cancel; // Cancela a execução
```

### Parallel Tasks

#### Execução Paralela de Múltiplas Tarefas

```delphi
var
  tasks: TArray<IOmniTask>;

tasks := [
  CreateTask(Worker1).Run,
  CreateTask(Worker2).Run,
  CreateTask(Worker3).Run
];

// Aguardar todas as tarefas
WaitForAll(tasks);
```

### Join

#### Sincronização de Tarefas

```delphi
var
  task1, task2: IOmniTask;

task1 := CreateTask(Worker1).Run;
task2 := CreateTask(Worker2).Run;

// Aguardar ambas as tarefas
Join([task1, task2]).WaitFor(INFINITE);
```

### Map

#### Transformação Paralela de Dados

```delphi
var
  input, output: TArray<integer>;
  result: TArray<string>;

input := [1, 2, 3, 4, 5];

result := Parallel.Map<integer, string>(
  input,
  function(const item: integer): string
  begin
    Result := IntToStr(item * 2);
  end
);
```

### ForEach

#### Iteração Paralela

```delphi
var
  items: TArray<integer>;

items := [1, 2, 3, 4, 5];

Parallel.ForEach<integer>(
  items,
  procedure(const item: integer)
  begin
    ProcessItem(item);
  end
);
```

### Pipeline

#### Processamento em Pipeline

```delphi
var
  pipeline: IOmniPipeline<integer, string>;

pipeline := Parallel.Pipeline<integer, string>
  .Stage(
    function(const input: integer): integer
    begin
      Result := input * 2;
    end
  )
  .Stage(
    function(const input: integer): string
    begin
      Result := IntToStr(input);
    end
  );

pipeline.Input.AddRange([1, 2, 3, 4, 5]);
pipeline.Run;

// Processar resultados
while not pipeline.Output.IsCompleted do begin
  if pipeline.Output.TryTake(item) then
    ProcessResult(item);
end;
```

### Fork/Join

#### Padrão Fork/Join para Divisão de Trabalho

```delphi
function CalculateSum(const items: TArray<integer>): integer;
begin
  if Length(items) <= 2 then begin
    // Caso base: calcular diretamente
    Result := 0;
    for var item in items do
      Result := Result + item;
  end else begin
    // Dividir e conquistar
    var mid := Length(items) div 2;
    var left := Copy(items, 0, mid);
    var right := Copy(items, mid, Length(items) - mid);
    
    var leftFuture := Future<integer>(
      function: integer
      begin
        Result := CalculateSum(left);
      end
    );
    
    var rightSum := CalculateSum(right);
    Result := leftFuture.Value + rightSum;
  end;
end;
```

### Background Worker

#### Worker de Background com Comunicação

```delphi
type
  TMyWorker = class(TOmniWorker)
  protected
    procedure DoWork; override;
  end;

procedure TMyWorker.DoWork;
begin
  while not CancellationToken.IsSignalled do begin
    // Fazer trabalho
    DoSomeWork;
    
    // Enviar progresso
    Task.Comm.Send(MSG_PROGRESS, GetProgress);
    
    // Verificar cancelamento
    if CancellationToken.IsSignalled then
      Break;
  end;
end;

// Uso
var
  worker: IOmniTask;

worker := CreateTask(TMyWorker.Create)
  .OnMessage(
    procedure(const task: IOmniTask; const msg: TOmniMessage)
    begin
      if msg.MsgID = MSG_PROGRESS then
        UpdateProgress(msg.MsgData.AsInteger);
    end
  )
  .Run;
```

### Data Flow

#### Fluxo de Dados com Transformações

```delphi
var
  dataFlow: IOmniDataFlow<integer, string>;

dataFlow := Parallel.DataFlow<integer, string>
  .FromSource(MyDataSource)
  .Transform(
    function(const input: integer): string
    begin
      Result := IntToStr(input * 2);
    end
  )
  .ToDestination(MyDataDestination);

dataFlow.Run;
```

### Data Manager

#### Gerenciamento de Dados Compartilhados

```delphi
var
  dataManager: IOmniDataManager;

dataManager := CreateDataManager;

// Registrar dados
dataManager.Register('config', MyConfig);
dataManager.Register('cache', MyCache);

// Acessar dados
var config := dataManager.Get('config') as TMyConfig;
```

### Output Queue

#### Fila de Saída para Resultados

```delphi
var
  outputQueue: IOmniOutputQueue<string>;

outputQueue := CreateOutputQueue<string>;

// Producer
for i := 1 to 100 do
  outputQueue.Add('Item ' + IntToStr(i));

// Consumer
while not outputQueue.IsCompleted do begin
  if outputQueue.TryTake(item) then
    ProcessItem(item);
end;
```

### Timed Task

#### Tarefa com Execução Temporizada

```delphi
var
  timedTask: IOmniTimedTask;

timedTask := CreateTimedTask(
  procedure
  begin
    DoPeriodicWork;
  end,
  1000 // Executar a cada 1000ms
);

timedTask.Run;
```

### Aggregation

#### Agregação de Resultados

```delphi
var
  results: TArray<integer>;
  sum: integer;

results := [1, 2, 3, 4, 5];

sum := Parallel.Aggregate<integer>(
  results,
  0, // Valor inicial
  function(const acc, item: integer): integer
  begin
    Result := acc + item;
  end
);
```

---

## Multi-threading de Baixo Nível

### Introdução

O nível baixo do OmniThreadLibrary oferece controle granular sobre threads e sincronização, ideal para casos onde o alto nível não é suficiente.

### Thread Management

#### Criação de Threads

```delphi
var
  thread: IOmniThread;

thread := CreateThread(
  procedure
  begin
    // Código da thread
  end
);

thread.Start;
```

#### Controle de Threads

```delphi
thread.Suspend;  // Suspender
thread.Resume;   // Retomar
thread.Terminate; // Terminar
thread.WaitFor;   // Aguardar conclusão
```

### Worker Classes

#### TOmniWorker

Classe base para workers customizados:

```delphi
type
  TMyWorker = class(TOmniWorker)
  protected
    procedure DoWork; override;
    procedure DoCleanup; override;
  end;

procedure TMyWorker.DoWork;
begin
  while not CancellationToken.IsSignalled do begin
    // Trabalho principal
    DoSomeWork;
    
    // Verificar cancelamento
    if CancellationToken.IsSignalled then
      Break;
  end;
end;

procedure TMyWorker.DoCleanup;
begin
  // Limpeza antes da destruição
  CleanupResources;
end;
```

#### TOmniWorker<T>

Worker genérico com tipo específico:

```delphi
type
  TStringWorker = class(TOmniWorker<string>)
  protected
    procedure DoWork; override;
  end;

procedure TStringWorker.DoWork;
begin
  // Trabalhar com dados do tipo string
  ProcessStringData(Data);
end;
```

### Communication

#### Message Passing

```delphi
// Enviar mensagem
Task.Comm.Send(MSG_DATA, 'Hello World');

// Receber mensagem
procedure HandleMessage(const task: IOmniTask; const msg: TOmniMessage);
begin
  case msg.MsgID of
    MSG_DATA: ProcessData(msg.MsgData.AsString);
  end;
end;
```

#### Channels

```delphi
var
  channel: IOmniChannel<integer>;

channel := CreateChannel<integer>;

// Producer
channel.Send(42);

// Consumer
var value: integer;
if channel.Receive(value) then
  ProcessValue(value);
```

### Synchronization Primitives

#### Events

```delphi
var
  event: IOmniEvent;

event := CreateEvent;

// Thread 1
event.Signal; // Sinalizar evento

// Thread 2
event.WaitFor(INFINITE); // Aguardar evento
```

#### Semaphores

```delphi
var
  semaphore: IOmniSemaphore;

semaphore := CreateSemaphore(3); // Máximo 3 recursos

// Adquirir recurso
semaphore.Acquire;

try
  // Usar recurso
  UseResource;
finally
  // Liberar recurso
  semaphore.Release;
end;
```

#### Mutexes

```delphi
var
  mutex: IOmniMutex;

mutex := CreateMutex;

// Thread 1
mutex.Acquire;
try
  // Seção crítica
  CriticalSection;
finally
  mutex.Release;
end;
```

### Collections

#### Thread-Safe Collections

```delphi
var
  list: IOmniList<integer>;
  queue: IOmniQueue<string>;
  stack: IOmniStack<boolean>;

list := CreateList<integer>;
queue := CreateQueue<string>;
stack := CreateStack<boolean>;

// Operações thread-safe
list.Add(42);
queue.Enqueue('Hello');
stack.Push(True);
```

### Performance Considerations

#### Thread Pooling

```delphi
var
  pool: IOmniThreadPool;

pool := CreateThreadPool('MyPool');
pool.MaxExecuting := 4; // Máximo 4 threads
pool.MinExecuting := 1; // Mínimo 1 thread
```

#### Memory Management

```delphi
// Usar interfaces para gerenciamento automático de memória
var
  task: IOmniTask; // Liberado automaticamente

// Evitar vazamentos de memória
task := CreateTask(MyWorker).Run;
// task será liberado quando sair de escopo
```

---

## Sincronização

### Introdução

A sincronização é fundamental para programação multi-thread segura. OmniThreadLibrary oferece várias primitivas de sincronização.

### Critical Sections

#### TOmniCS

Seção crítica simples e eficiente:

```delphi
var
  cs: TOmniCS;
  data: integer;

cs.Initialize;

// Thread 1
cs.Acquire;
try
  Inc(data);
finally
  cs.Release;
end;

// Thread 2
cs.Acquire;
try
  Dec(data);
finally
  cs.Release;
end;
```

#### RAII Pattern

```delphi
var
  cs: TOmniCS;
  lock: TOmniCSLock;

cs.Initialize;

// Lock automático
lock := cs.Lock; // Adquire automaticamente
try
  Inc(data);
finally
  // Libera automaticamente quando sai do escopo
end;
```

### Events

#### Manual Reset Event

```delphi
var
  event: IOmniEvent;

event := CreateEvent(True); // Manual reset

// Thread 1
event.Signal; // Sinalizar

// Thread 2
event.WaitFor(INFINITE); // Aguardar
```

#### Auto Reset Event

```delphi
var
  event: IOmniEvent;

event := CreateEvent(False); // Auto reset

// Thread 1
event.Signal; // Sinalizar (reset automático)

// Thread 2
event.WaitFor(INFINITE); // Aguardar
```

### Semaphores

#### Controle de Recursos

```delphi
var
  semaphore: IOmniSemaphore;

semaphore := CreateSemaphore(3); // Máximo 3 recursos

// Adquirir recurso
semaphore.Acquire;

try
  // Usar recurso
  UseResource;
finally
  // Liberar recurso
  semaphore.Release;
end;
```

### Mutexes

#### Exclusão Mútua

```delphi
var
  mutex: IOmniMutex;

mutex := CreateMutex;

// Thread 1
mutex.Acquire;
try
  // Seção crítica
  CriticalSection;
finally
  mutex.Release;
end;
```

### Reader-Writer Locks

#### Múltiplos Leitores, Escritor Único

```delphi
var
  rwLock: IOmniReaderWriterLock;

rwLock := CreateReaderWriterLock;

// Leitores
rwLock.AcquireRead;
try
  ReadData;
finally
  rwLock.ReleaseRead;
end;

// Escritor
rwLock.AcquireWrite;
try
  WriteData;
finally
  rwLock.ReleaseWrite;
end;
```

### Barriers

#### Sincronização de Múltiplas Threads

```delphi
var
  barrier: IOmniBarrier;

barrier := CreateBarrier(3); // 3 threads

// Thread 1
DoWork1;
barrier.SignalAndWait; // Aguarda outras threads

// Thread 2
DoWork2;
barrier.SignalAndWait; // Aguarda outras threads

// Thread 3
DoWork3;
barrier.SignalAndWait; // Todas as threads continuam
```

### Countdown Events

#### Aguardar Múltiplas Operações

```delphi
var
  countdown: IOmniCountdownEvent;

countdown := CreateCountdownEvent(3); // Aguardar 3 operações

// Thread 1
DoWork1;
countdown.Signal; // Uma operação concluída

// Thread 2
DoWork2;
countdown.Signal; // Duas operações concluídas

// Thread 3
DoWork3;
countdown.Signal; // Todas as operações concluídas

// Thread principal
countdown.WaitFor(INFINITE); // Aguarda todas as operações
```

### Spin Locks

#### Locks de Alta Performance

```delphi
var
  spinLock: TOmniSpinLock;
  data: integer;

spinLock.Initialize;

// Thread 1
spinLock.Acquire;
try
  Inc(data);
finally
  spinLock.Release;
end;
```

### Lock-Free Programming

#### Operações Atômicas

```delphi
var
  atomicInt: TOmniAlignedInt32;

atomicInt := 0;

// Thread 1
atomicInt.Increment;

// Thread 2
atomicInt.Decrement;

// Thread 3
var value := atomicInt.Value; // Leitura atômica
```

#### Compare and Swap

```delphi
var
  atomicInt: TOmniAlignedInt32;

atomicInt := 0;

// Compare and swap
if atomicInt.CompareExchange(1, 0) then
  ShowMessage('Valor alterado de 0 para 1');
```

### Best Practices

#### 1. Evitar Deadlocks

```delphi
// ERRADO: Pode causar deadlock
lock1.Acquire;
lock2.Acquire;
// ...

// CORRETO: Ordem consistente
lock1.Acquire;
try
  lock2.Acquire;
  try
    // ...
  finally
    lock2.Release;
  end;
finally
  lock1.Release;
end;
```

#### 2. Timeout em Operações

```delphi
if cs.Acquire(1000) then // Timeout de 1 segundo
try
  // Seção crítica
finally
  cs.Release;
end
else
  ShowMessage('Timeout na aquisição do lock');
```

#### 3. Uso de RAII

```delphi
var
  cs: TOmniCS;
  lock: TOmniCSLock;

cs.Initialize;

lock := cs.Lock; // Adquire automaticamente
try
  // Seção crítica
finally
  // Libera automaticamente
end;
```

---

## Recursos Diversos

### Introdução

Esta seção cobre recursos adicionais e utilitários do OmniThreadLibrary.

### Utilities

#### Timer Utilities

```delphi
var
  timer: IOmniTimer;

timer := CreateTimer(
  procedure
  begin
    DoPeriodicWork;
  end,
  1000 // Intervalo de 1000ms
);

timer.Start;
```

#### Delay Utilities

```delphi
// Delay não-bloqueante
Delay(1000); // Aguarda 1 segundo

// Delay com cancelamento
if Delay(1000, cancellationToken) then
  ShowMessage('Delay concluído')
else
  ShowMessage('Delay cancelado');
```

### Monitoring

#### Task Monitoring

```delphi
var
  monitor: IOmniTaskMonitor;

monitor := CreateTaskMonitor;

// Registrar tarefa
monitor.Register(task);

// Obter estatísticas
var stats := monitor.GetStatistics(task);
ShowMessage('CPU: ' + FloatToStr(stats.CPUUsage));
```

#### Performance Counters

```delphi
var
  counter: IOmniPerformanceCounter;

counter := CreatePerformanceCounter('MyCounter');

counter.Start;
// Código a ser medido
counter.Stop;

ShowMessage('Tempo: ' + FloatToStr(counter.ElapsedMilliseconds));
```

### Debugging

#### Debug Helpers

```delphi
// Habilitar debug
OmniThreadLibrary.DebugMode := True;

// Log de debug
OmniThreadLibrary.DebugLog('Mensagem de debug');

// Breakpoint em tasks
OmniThreadLibrary.DebugBreakOnTaskCreation := True;
```

#### Exception Handling

```delphi
var
  task: IOmniTask;

task := CreateTask(
  procedure
  begin
    try
      RiskyOperation;
    except
      on E: Exception do
        Task.Comm.Send(MSG_EXCEPTION, E.Message);
    end;
  end
)
.OnMessage(
  procedure(const task: IOmniTask; const msg: TOmniMessage)
  begin
    if msg.MsgID = MSG_EXCEPTION then
      LogError(msg.MsgData.AsString);
  end
)
.Run;
```

### Configuration

#### Global Configuration

```delphi
// Configurar pool global
OmniThreadLibrary.GlobalThreadPool.MaxExecuting := 8;
OmniThreadLibrary.GlobalThreadPool.MinExecuting := 2;

// Configurar timeout padrão
OmniThreadLibrary.DefaultTimeout := 5000; // 5 segundos
```

#### Task Configuration

```delphi
var
  config: IOmniTaskConfig;

config := CreateTaskConfig
  .Name('MyTask')
  .Priority(tpHigh)
  .Affinity([0, 1]) // Usar apenas CPUs 0 e 1
  .SetParameter('param1', 'value1');

var task := CreateTask(MyWorker).Configure(config).Run;
```

### Memory Management

#### Smart Pointers

```delphi
var
  smartPtr: IOmniSmartPtr<TMyObject>;

smartPtr := CreateSmartPtr<TMyObject>(TMyObject.Create);

// Uso automático de referência
var obj := smartPtr.Value;
```

#### Memory Pools

```delphi
var
  pool: IOmniMemoryPool;

pool := CreateMemoryPool(1024, 100); // Blocos de 1024 bytes, 100 blocos

var ptr := pool.Allocate;
try
  // Usar memória
finally
  pool.Deallocate(ptr);
end;
```

### Serialization

#### Data Serialization

```delphi
var
  serializer: IOmniSerializer;

serializer := CreateSerializer;

// Serializar objeto
var data := serializer.Serialize(myObject);

// Deserializar objeto
var obj := serializer.Deserialize<TMyObject>(data);
```

### Networking

#### TCP Communication

```delphi
var
  tcpClient: IOmniTcpClient;

tcpClient := CreateTcpClient('localhost', 8080);

tcpClient.Connect;
try
  tcpClient.Send('Hello Server');
  var response := tcpClient.Receive;
finally
  tcpClient.Disconnect;
end;
```

#### UDP Communication

```delphi
var
  udpClient: IOmniUdpClient;

udpClient := CreateUdpClient('localhost', 8080);

udpClient.Send('Hello Server');
var response := udpClient.Receive;
```

---

## Guias Práticos

### Introdução

Esta seção contém guias práticos para cenários comuns de uso do OmniThreadLibrary.

### File Processing

#### Processamento Paralelo de Arquivos

```delphi
procedure ProcessFilesInParallel(const filePaths: TArray<string>);
var
  tasks: TArray<IOmniTask>;
begin
  SetLength(tasks, Length(filePaths));
  
  for var i := 0 to High(filePaths) do
    tasks[i] := CreateTask(
      procedure(const filePath: string)
      begin
        ProcessFile(filePath);
      end,
      filePaths[i]
    ).Run;
  
  WaitForAll(tasks);
end;
```

#### File Search

```delphi
procedure SearchFilesInParallel(const searchPath: string; const pattern: string);
var
  files: TArray<string>;
  results: TArray<string>;
begin
  files := GetFiles(searchPath, pattern);
  
  results := Parallel.Map<string, string>(
    files,
    function(const filePath: string): string
    begin
      if SearchInFile(filePath, searchTerm) then
        Result := filePath
      else
        Result := '';
    end
  );
  
  // Filtrar resultados vazios
  results := FilterEmpty(results);
end;
```

### Database Operations

#### Connection Pooling

```delphi
type
  TDatabaseWorker = class(TOmniWorker)
  private
    FConnection: TADOConnection;
  protected
    procedure DoWork; override;
  public
    constructor Create(const connectionString: string);
    destructor Destroy; override;
  end;

constructor TDatabaseWorker.Create(const connectionString: string);
begin
  inherited Create;
  FConnection := TADOConnection.Create(nil);
  FConnection.ConnectionString := connectionString;
  FConnection.Open;
end;

procedure TDatabaseWorker.DoWork;
begin
  while not CancellationToken.IsSignalled do begin
    var query := GetNextQuery;
    if query <> '' then begin
      ExecuteQuery(FConnection, query);
      Task.Comm.Send(MSG_QUERY_COMPLETED, query);
    end;
  end;
end;
```

#### Batch Processing

```delphi
procedure ProcessDatabaseBatch(const queries: TArray<string>);
var
  batchSize: integer;
  batches: TArray<TArray<string>>;
begin
  batchSize := 10;
  batches := SplitIntoBatches(queries, batchSize);
  
  Parallel.ForEach<TArray<string>>(
    batches,
    procedure(const batch: TArray<string>)
    begin
      var connection := CreateConnection;
      try
        for var query in batch do
          ExecuteQuery(connection, query);
      finally
        connection.Free;
      end;
    end
  );
end;
```

### Web Scraping

#### Parallel Web Requests

```delphi
procedure ScrapeWebsitesInParallel(const urls: TArray<string>);
var
  results: TArray<string>;
begin
  results := Parallel.Map<string, string>(
    urls,
    function(const url: string): string
    begin
      var httpClient := THTTPClient.Create;
      try
        Result := httpClient.Get(url).ContentAsString;
      finally
        httpClient.Free;
      end;
    end
  );
end;
```

#### Rate Limiting

```delphi
procedure ScrapeWithRateLimit(const urls: TArray<string>);
var
  semaphore: IOmniSemaphore;
begin
  semaphore := CreateSemaphore(5); // Máximo 5 requests simultâneos
  
  Parallel.ForEach<string>(
    urls,
    procedure(const url: string)
    begin
      semaphore.Acquire;
      try
        var httpClient := THTTPClient.Create;
        try
          var content := httpClient.Get(url).ContentAsString;
          ProcessContent(content);
        finally
          httpClient.Free;
        end;
        
        Sleep(1000); // Rate limit de 1 segundo
      finally
        semaphore.Release;
      end;
    end
  );
end;
```

### Image Processing

#### Parallel Image Processing

```delphi
procedure ProcessImagesInParallel(const imagePaths: TArray<string>);
begin
  Parallel.ForEach<string>(
    imagePaths,
    procedure(const imagePath: string)
    begin
      var bitmap := TBitmap.Create;
      try
        bitmap.LoadFromFile(imagePath);
        
        // Processar imagem
        ResizeImage(bitmap, 800, 600);
        ApplyFilter(bitmap);
        
        // Salvar resultado
        var outputPath := ChangeFileExt(imagePath, '_processed.jpg');
        bitmap.SaveToFile(outputPath);
      finally
        bitmap.Free;
      end;
    end
  );
end;
```

#### Image Pipeline

```delphi
procedure ProcessImagesWithPipeline(const imagePaths: TArray<string>);
var
  pipeline: IOmniPipeline<string, TBitmap>;
begin
  pipeline := Parallel.Pipeline<string, TBitmap>
    .Stage(
      function(const imagePath: string): TBitmap
      begin
        Result := TBitmap.Create;
        Result.LoadFromFile(imagePath);
      end
    )
    .Stage(
      function(const bitmap: TBitmap): TBitmap
      begin
        ResizeImage(bitmap, 800, 600);
        Result := bitmap;
      end
    )
    .Stage(
      function(const bitmap: TBitmap): TBitmap
      begin
        ApplyFilter(bitmap);
        Result := bitmap;
      end
    );
  
  pipeline.Input.AddRange(imagePaths);
  pipeline.Run;
  
  // Processar resultados
  while not pipeline.Output.IsCompleted do begin
    if pipeline.Output.TryTake(bitmap) then begin
      var outputPath := GenerateOutputPath;
      bitmap.SaveToFile(outputPath);
      bitmap.Free;
    end;
  end;
end;
```

### Data Analysis

#### Parallel Data Processing

```delphi
procedure AnalyzeDataInParallel(const data: TArray<TDataRecord>);
var
  results: TArray<TAnalysisResult>;
begin
  results := Parallel.Map<TDataRecord, TAnalysisResult>(
    data,
    function(const record: TDataRecord): TAnalysisResult
    begin
      Result := AnalyzeRecord(record);
    end
  );
  
  // Agregar resultados
  var finalResult := AggregateResults(results);
  DisplayResults(finalResult);
end;
```

#### Statistical Analysis

```delphi
procedure CalculateStatisticsInParallel(const data: TArray<Double>);
var
  tasks: TArray<IOmniTask>;
  mean, median, stdDev: Double;
begin
  SetLength(tasks, 3);
  
  // Calcular média
  tasks[0] := CreateTask(
    function: Double
    begin
      Result := CalculateMean(data);
    end
  ).Run;
  
  // Calcular mediana
  tasks[1] := CreateTask(
    function: Double
    begin
      Result := CalculateMedian(data);
    end
  ).Run;
  
  // Calcular desvio padrão
  tasks[2] := CreateTask(
    function: Double
    begin
      Result := CalculateStandardDeviation(data);
    end
  ).Run;
  
  // Aguardar resultados
  mean := (tasks[0] as IOmniFuture<Double>).Value;
  median := (tasks[1] as IOmniFuture<Double>).Value;
  stdDev := (tasks[2] as IOmniFuture<Double>).Value;
  
  DisplayStatistics(mean, median, stdDev);
end;
```

### GUI Applications

#### Background Tasks in GUI

```delphi
type
  TMainForm = class(TForm)
  private
    FBackgroundTask: IOmniTask;
    procedure HandleTaskMessage(const task: IOmniTask; const msg: TOmniMessage);
    procedure HandleTaskTerminated(const task: IOmniTask);
  public
    procedure StartBackgroundWork;
  end;

procedure TMainForm.StartBackgroundWork;
begin
  FBackgroundTask := CreateTask(
    procedure
    begin
      // Trabalho pesado em background
      DoHeavyWork;
      
      // Enviar resultado para UI thread
      Task.Comm.Send(MSG_WORK_COMPLETED, 'Trabalho concluído!');
    end
  )
  .OnMessage(HandleTaskMessage)
  .OnTerminated(HandleTaskTerminated)
  .Run;
end;

procedure TMainForm.HandleTaskMessage(const task: IOmniTask; const msg: TOmniMessage);
begin
  case msg.MsgID of
    MSG_WORK_COMPLETED:
      ShowMessage(msg.MsgData.AsString);
    MSG_PROGRESS:
      ProgressBar1.Position := msg.MsgData.AsInteger;
  end;
end;

procedure TMainForm.HandleTaskTerminated(const task: IOmniTask);
begin
  FBackgroundTask := nil;
  Button1.Enabled := True;
end;
```

#### Progress Reporting

```delphi
procedure DoWorkWithProgress(const totalItems: integer);
var
  task: IOmniTask;
  processedItems: integer;
begin
  processedItems := 0;
  
  task := CreateTask(
    procedure
    begin
      for var i := 1 to totalItems do begin
        // Processar item
        ProcessItem(i);
        
        // Atualizar progresso
        Inc(processedItems);
        Task.Comm.Send(MSG_PROGRESS, processedItems);
        
        // Verificar cancelamento
        if CancellationToken.IsSignalled then
          Break;
      end;
    end
  )
  .OnMessage(
    procedure(const task: IOmniTask; const msg: TOmniMessage)
    begin
      if msg.MsgID = MSG_PROGRESS then begin
        var progress := msg.MsgData.AsInteger;
        ProgressBar1.Position := (progress * 100) div totalItems;
        Label1.Caption := Format('Processados: %d de %d', [progress, totalItems]);
      end;
    end
  )
  .Run;
end;
```

### Error Handling

#### Robust Error Handling

```delphi
procedure RobustParallelProcessing(const items: TArray<string>);
begin
  Parallel.ForEach<string>(
    items,
    procedure(const item: string)
    begin
      try
        ProcessItem(item);
      except
        on E: Exception do begin
          // Log do erro
          LogError(Format('Erro ao processar %s: %s', [item, E.Message]));
          
          // Continuar processamento
          // Não re-raise a exceção
        end;
      end;
    end
  );
end;
```

#### Retry Logic

```delphi
procedure ProcessWithRetry(const item: string; maxRetries: integer);
var
  retryCount: integer;
  success: boolean;
begin
  retryCount := 0;
  success := False;
  
  while (retryCount < maxRetries) and not success do begin
    try
      ProcessItem(item);
      success := True;
    except
      on E: Exception do begin
        Inc(retryCount);
        if retryCount < maxRetries then begin
          LogWarning(Format('Tentativa %d falhou para %s: %s', [retryCount, item, E.Message]));
          Sleep(1000 * retryCount); // Backoff exponencial
        end else begin
          LogError(Format('Falha final para %s após %d tentativas: %s', [item, maxRetries, E.Message]));
        end;
      end;
    end;
  end;
end;
```

---

## Apêndices

### A. Units

#### Units Principais
- **OtlCommon**: Funcionalidades comuns
- **OtlSync**: Primitivas de sincronização
- **OtlComm**: Comunicação entre threads
- **OtlTask**: Gerenciamento de tasks
- **OtlParallel**: Operações paralelas
- **OtlCollections**: Coleções thread-safe
- **OtlContainers**: Containers especializados
- **OtlDataStructs**: Estruturas de dados
- **OtlHooks**: Sistema de hooks
- **OtlEventMonitor**: Monitoramento de eventos

#### Units de Suporte
- **OtlUtils**: Utilitários diversos
- **OtlStreams**: Streams thread-safe
- **OtlFileUtils**: Utilitários de arquivo
- **OtlStringUtils**: Utilitários de string
- **OtlDateTimeUtils**: Utilitários de data/hora

### B. Demo Applications

#### Demos Incluídos
1. **BasicDemo**: Demonstração básica de tasks
2. **CommunicationDemo**: Comunicação entre threads
3. **PipelineDemo**: Processamento em pipeline
4. **ParallelDemo**: Operações paralelas
5. **SynchronizationDemo**: Primitivas de sincronização
6. **PerformanceDemo**: Testes de performance
7. **ErrorHandlingDemo**: Tratamento de erros
8. **GUIDemo**: Integração com interface gráfica

#### Como Executar Demos
1. Abra o projeto demo no Delphi
2. Compile e execute
3. Experimente diferentes configurações
4. Analise o código fonte

### C. Examples

#### Exemplos de Código
1. **File Processing**: Processamento paralelo de arquivos
2. **Database Operations**: Operações de banco de dados
3. **Web Scraping**: Coleta de dados web
4. **Image Processing**: Processamento de imagens
5. **Data Analysis**: Análise de dados
6. **GUI Integration**: Integração com interface gráfica
7. **Error Handling**: Tratamento robusto de erros
8. **Performance Optimization**: Otimização de performance

### D. Hooking into OmniThreadLibrary

#### Sistema de Hooks
OmniThreadLibrary oferece um sistema de hooks para interceptar e modificar comportamento interno.

#### Tipos de Hooks
- **Task Creation Hooks**: Interceptar criação de tasks
- **Task Termination Hooks**: Interceptar término de tasks
- **Message Hooks**: Interceptar mensagens
- **Exception Hooks**: Interceptar exceções

#### Exemplo de Hook

```delphi
procedure RegisterHooks;
begin
  // Hook de criação de task
  OmniThreadLibrary.RegisterTaskCreationHook(
    procedure(const task: IOmniTask)
    begin
      LogInfo('Task criada: ' + task.Name);
    end
  );
  
  // Hook de término de task
  OmniThreadLibrary.RegisterTaskTerminationHook(
    procedure(const task: IOmniTask)
    begin
      LogInfo('Task terminada: ' + task.Name);
    end
  );
end;
```

### E. ForEach Internals

#### Implementação Interna
O `ForEach` é implementado usando um pool de threads e distribuição de trabalho.

#### Algoritmo de Distribuição
1. Dividir trabalho em chunks
2. Distribuir chunks para threads disponíveis
3. Processar chunks em paralelo
4. Coletar resultados

#### Otimizações
- **Work Stealing**: Threads podem "roubar" trabalho de outras threads
- **Load Balancing**: Balanceamento automático de carga
- **Memory Pooling**: Reutilização de memória

### F. Hyperlinks

#### Links Úteis
- **Site Oficial**: https://www.omnithreadlibrary.com/
- **GitHub**: https://github.com/gabr42/OmniThreadLibrary
- **Documentação**: https://www.omnithreadlibrary.com/book/
- **Fórum**: https://en.delphipraxis.net/forum/omnithreadlibrary/
- **StackOverflow**: https://stackoverflow.com/questions/tagged/omnithreadlibrary

#### Recursos Adicionais
- **Blog do Autor**: https://thedelphigeek.com/
- **Webinars**: Disponíveis no site oficial
- **Exemplos**: Incluídos na distribuição
- **Comunidade**: Fórum ativo de desenvolvedores

### G. Notes

#### Notas Importantes
1. **Thread Safety**: Sempre considere thread safety ao usar recursos compartilhados
2. **Memory Management**: Use interfaces para gerenciamento automático de memória
3. **Performance**: Monitore performance e ajuste configurações conforme necessário
4. **Error Handling**: Implemente tratamento robusto de erros
5. **Testing**: Teste extensivamente em diferentes cenários

#### Limitações Conhecidas
1. **Console Applications**: Requer message loop manual
2. **Cross-Platform**: Suporte limitado para plataformas não-Windows
3. **Memory Usage**: Pode usar mais memória que threads tradicionais
4. **Debugging**: Debugging pode ser mais complexo

#### Melhores Práticas
1. **Use High-Level APIs**: Prefira APIs de alto nível quando possível
2. **Avoid Locking**: Evite locks sempre que possível
3. **Use Messaging**: Prefira comunicação por mensagens
4. **Monitor Performance**: Monitore performance regularmente
5. **Handle Errors**: Implemente tratamento robusto de erros

---

## Conclusão

OmniThreadLibrary é uma biblioteca poderosa e flexível para programação multi-thread em Delphi. Com suas APIs de alto e baixo nível, oferece soluções para uma ampla gama de cenários de programação paralela.

### Principais Benefícios
- **Simplicidade**: APIs fáceis de usar
- **Performance**: Otimizada para máxima eficiência
- **Flexibilidade**: Suporte para diferentes níveis de abstração
- **Robustez**: Tratamento robusto de erros e exceções
- **Comunidade**: Ativa comunidade de desenvolvedores

### Quando Usar
- **Processamento Paralelo**: Quando você precisa processar dados em paralelo
- **Operações de I/O**: Para operações de entrada/saída não-bloqueantes
- **Cálculos Intensivos**: Para cálculos que podem ser paralelizados
- **Aplicações Responsivas**: Para manter interfaces responsivas

### Próximos Passos
1. **Experimente**: Comece com exemplos simples
2. **Leia a Documentação**: Estude a documentação completa
3. **Participe da Comunidade**: Junte-se ao fórum e StackOverflow
4. **Contribua**: Contribua com melhorias e exemplos

---

*Esta documentação foi criada com base no manual oficial da OmniThreadLibrary versão 3.07.7, garantindo que o Agent tenha total e pleno domínio e conhecimento desta biblioteca.*
