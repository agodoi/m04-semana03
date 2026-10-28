# Atendimento do professor

## Terças e quintas-feiras

Meu regime de trabalho é de horário parcial. Nesses dois dias, estarei disponível para atendê-los!

# Semana 03: Ponteiros, arrays e leitura de botão com debounce no ESP32

Nesta semana, estudaremos como variáveis e arrays são armazenados na memória e como utilizar ponteiros para acessar seus valores.

Também aprenderemos a identificar pinos adequados para conectar um botão ao ESP32 e a tratar o repique dos contatos mecânicos, conhecido como *bounce*. Essa leitura permitirá selecionar opções de um menu com um avanço por acionamento.

> **Placa de referência:** os exemplos de ligação desta aula consideram uma placa de desenvolvimento com ESP32 clássico, como a ESP32 DevKit com módulo ESP32-WROOM-32. Antes de montar o circuito, confira o pinout da sua placa. Outras versões da família ESP32 podem ter pinos e restrições diferentes.

## Impactos no seu projeto

Ao final das atividades, você deverá ser capaz de:

- Identificar endereços de variáveis e de elementos de arrays.
- Diferenciar um array de um ponteiro.
- Utilizar ponteiros para acessar e modificar valores.
- Percorrer um array por meio de um ponteiro.
- Armazenar leituras em um buffer circular e calcular sua média.
- Identificar um GPIO adequado para conectar um botão.
- Implementar debounce por software.
- Alterar a opção selecionada em um menu uma vez a cada pressão do botão.

---

## 1. Variáveis, arrays e memória

Uma variável representa um objeto armazenado na memória. Esse objeto possui um tipo, ocupa uma determinada quantidade de bytes e tem um endereço.

Observe:

```cpp
int x = 9;
```

Nesse exemplo:

- `int` é o tipo da variável.
- `x` é o nome da variável.
- `9` é o valor armazenado.
- `&x` permite obter o endereço de `x`.
- `sizeof(x)` informa quantos bytes `x` ocupa.

O tamanho de um tipo pode variar entre plataformas. Por isso, utilize `sizeof` para verificá-lo no ambiente em uso.

### Arrays

Um array reúne uma quantidade fixa de elementos do mesmo tipo, armazenados consecutivamente na memória.

```cpp
int valores[3] = {5, 10, 15};
```

Esse array contém três elementos:

| Índice | Elemento | Valor |
|---|---|---|
| 0 | `valores[0]` | 5 |
| 1 | `valores[1]` | 10 |
| 2 | `valores[2]` | 15 |

O primeiro índice é `0`. Portanto, um array de três elementos possui índices válidos de `0` a `2`.

> **Atenção:** acessar um elemento fora dos limites do array provoca comportamento indefinido. O programa pode apresentar resultados incorretos, falhar ou aparentar funcionar sem que o acesso seja válido.

---

## 2. Ponteiros

Ponteiros são variáveis que podem armazenar endereços de objetos na memória, como variáveis e elementos de arrays.

O tipo do ponteiro indica o tipo do objeto apontado.

```cpp
int x = 9;
int *pt = &x;
```

Nesse exemplo:

- `x` armazena o valor `9`.
- `&x` obtém o endereço de `x`.
- `pt` armazena esse endereço.
- `*pt` permite acessar o valor de `x`.

### Os operadores `&` e `*`

| Expressão | Significado neste exemplo |
|---|---|
| `x` | Valor armazenado em `x` |
| `&x` | Endereço de `x` |
| `pt` | Endereço armazenado no ponteiro |
| `&pt` | Endereço da própria variável ponteiro |
| `*pt` | Valor do objeto apontado |

Na declaração `int *pt`, o asterisco indica que `pt` é um ponteiro para `int`.

Em uma expressão como `*pt = 12`, o asterisco é o operador de desreferenciação: ele permite acessar o objeto apontado.

```cpp
int x = 9;
int *pt = &x;

*pt = 12;

// Agora x também vale 12.
```

### Cuidados com ponteiros

Antes de desreferenciar um ponteiro, ele deve apontar para um objeto válido e compatível com seu tipo.

Não desreferencie ponteiros:

- Não inicializados.
- Nulos.
- Que apontem para objetos que já deixaram de existir.
- Que estejam fora dos limites permitidos de um array.

### Material de apoio

Consulte a [planilha de apoio sobre memória e ponteiros](https://docs.google.com/spreadsheets/d/1zgsKqh5UbhyGN3BkQBueWcM0Ubp_ikUC9deWb7eIvBs/edit?usp=sharing).

---

## 3. Práticas com ponteiros

Os exemplos seguintes utilizam o ambiente Arduino para ESP32.

Para exibir endereços, utilizaremos:

```cpp
Serial.printf("%p\n", (void *)&variavel);
```

O especificador `%p` permite apresentar um ponteiro. A conversão para `void*` fornece o tipo esperado por esse especificador.

> Os endereços exibidos podem mudar entre compilações e configurações. O objetivo da prática é interpretar o significado de cada endereço, e não memorizar seus valores.

### Exemplo 1 — Endereço e tamanho de uma variável

```cpp
char x = 'A';

void setup() {
    Serial.begin(115200);

    Serial.printf("Endereco de x: %p\n", (void *)&x);

    Serial.print("Valor de x: ");
    Serial.println(x);

    Serial.print("Tamanho de x em bytes: ");
    Serial.println(sizeof(x));
}

void loop() {
}
```

Observe que o endereço, o valor e o tamanho da variável são informações diferentes.

### Exemplo 2 — Endereços de variáveis independentes

```cpp
char x = 'A';
char y = 'B';
char z = 'C';

void setup() {
    Serial.begin(115200);

    Serial.printf("Endereco de x: %p\n", (void *)&x);
    Serial.printf("Endereco de y: %p\n", (void *)&y);
    Serial.printf("Endereco de z: %p\n", (void *)&z);
}

void loop() {
}
```

As variáveis `x`, `y` e `z` são objetos independentes.

Mesmo que os endereços apareçam próximos em uma execução, não devemos concluir que variáveis independentes serão sempre armazenadas consecutivamente ou na ordem em que foram declaradas.

### Exemplo 3 — Endereços dos elementos de um array

```cpp
char xyz[3] = {'x', 'y', 'z'};

void setup() {
    Serial.begin(115200);

    Serial.printf("Endereco de xyz[0]: %p\n", (void *)&xyz[0]);
    Serial.printf("Endereco de xyz[1]: %p\n", (void *)&xyz[1]);
    Serial.printf("Endereco de xyz[2]: %p\n", (void *)&xyz[2]);

    Serial.print("Tamanho de cada elemento em bytes: ");
    Serial.println(sizeof(xyz[0]));

    Serial.print("Tamanho total do array em bytes: ");
    Serial.println(sizeof(xyz));
}

void loop() {
}
```

Diferentemente de variáveis independentes, os elementos de um array são armazenados consecutivamente.

Neste exemplo, cada elemento é do tipo `char`, cujo tamanho é um byte em C++.

> Esse array contém três caracteres, mas não possui o terminador `'\0'`. Portanto, não deve ser tratado como uma string C terminada em zero.

### Array e ponteiro são diferentes

Considere:

```cpp
int valores[3] = {5, 10, 15};
int *pt = valores;
```

Nesse contexto, `valores` é convertido em um ponteiro para o primeiro elemento do array. Assim, a inicialização equivale a:

```cpp
int *pt = &valores[0];
```

Entretanto, o array continua sendo um conjunto de três elementos, enquanto `pt` é uma variável que armazena um endereço.

No exemplo com `char xyz[3]`:

| Expressão | Tipo | Significado |
|---|---|---|
| `&xyz[0]` | `char*` | Ponteiro para o primeiro elemento |
| `&xyz` | `char (*)[3]` | Ponteiro para o array inteiro |

Essas expressões representam o mesmo endereço inicial quando exibidas como `void*`, mas possuem tipos diferentes.

Por isso:

- `&xyz[0] + 1` aponta para o segundo elemento.
- `&xyz + 1` aponta para a posição imediatamente após o array inteiro.

A posição imediatamente após um array pode ser utilizada como limite de percurso, mas não pode ser desreferenciada.

### Exemplo 4 — Endereço do ponteiro, endereço armazenado e valor apontado

```cpp
int xyz[3] = {222, 333, 444};
int *pt = &xyz[2];

void setup() {
    Serial.begin(115200);

    Serial.printf("Endereco da variavel pt: %p\n", (void *)&pt);
    Serial.printf("Endereco armazenado em pt: %p\n", (void *)pt);
    Serial.printf("Endereco de xyz[2]: %p\n", (void *)&xyz[2]);

    Serial.print("Valor apontado por pt: ");
    Serial.println(*pt);

    *pt = 555;

    Serial.print("Novo valor de xyz[2]: ");
    Serial.println(xyz[2]);
}

void loop() {
}
```

Antes da alteração, `*pt` vale `444`.

Após executar `*pt = 555`, o elemento `xyz[2]` passa a valer `555`, pois o ponteiro permite acessar esse elemento.

### Aritmética de ponteiros

Considere:

```cpp
int valores[3] = {5, 10, 15};
int *pt = valores;
```

As expressões abaixo acessam os mesmos elementos:

| Acesso por índice | Acesso por ponteiro | Valor |
|---|---|---|
| `valores[0]` | `*(pt + 0)` | 5 |
| `valores[1]` | `*(pt + 1)` | 10 |
| `valores[2]` | `*(pt + 2)` | 15 |

Somar `1` a um ponteiro avança um elemento do tipo apontado. Não significa, necessariamente, avançar um byte.

A aritmética deve respeitar os limites do array. Não utilize essa operação para tentar percorrer variáveis independentes.

---

## 4. Exemplo 5 — Leituras de temperatura e média móvel

Nesta prática, armazenaremos até cinco leituras válidas de temperatura em um array.

Quando todas as posições estiverem ocupadas, cada nova leitura substituirá a mais antiga. Essa organização é chamada de **buffer circular**.

Calcularemos a média das leituras armazenadas para suavizar variações entre amostras.

### Preparação

No Gerenciador de Bibliotecas do Arduino IDE, instale:

- **DHT sensor library**, da Adafruit.
- **Adafruit Unified Sensor**, dependência da biblioteca.

Para a montagem de referência:

- Utilize o GPIO 4 para o sinal do DHT22.
- Utilize alimentação e níveis de sinal compatíveis com o ESP32.
- Conecte os terras em comum.
- Confira a pinagem do sensor ou do módulo utilizado.
- Se o sensor não possuir resistor de pull-up integrado, utilize o resistor indicado em sua documentação, mantendo o sinal compatível com 3,3 V.

> O GPIO 4 é a escolha deste exemplo para a placa de referência. Confira sua disponibilidade na placa utilizada.

### Código

```cpp
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

const unsigned long intervaloLeitura = 2000;
unsigned long ultimoTempoLeitura = 0;

const int numLeituras = 5;
float temperaturas[numLeituras] = {0};

int indice = 0;
int quantidadeValidas = 0;

// Pre-condicoes:
// - leituras aponta para pelo menos "tamanho" elementos validos;
// - tamanho e maior que zero.
float calcularMedia(const float *leituras, int tamanho) {
    float soma = 0.0f;

    for (int i = 0; i < tamanho; i++) {
        soma += *(leituras + i);
    }

    return soma / tamanho;
}

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void loop() {
    unsigned long tempoAtual = millis();

    if (tempoAtual - ultimoTempoLeitura >= intervaloLeitura) {
        ultimoTempoLeitura = tempoAtual;

        float temperatura = dht.readTemperature();

        if (isnan(temperatura)) {
            Serial.println("Falha na leitura do sensor.");
            return;
        }

        temperaturas[indice] = temperatura;

        // Avanca circularmente a posicao de escrita.
        indice = (indice + 1) % numLeituras;

        if (quantidadeValidas < numLeituras) {
            quantidadeValidas++;
        }

        float media =
            calcularMedia(temperaturas, quantidadeValidas);

        Serial.print("Media de temperatura (");
        Serial.print(quantidadeValidas);
        Serial.print(" amostras): ");
        Serial.print(media, 2);
        Serial.println(" graus Celsius");
    }

    // Outras tarefas podem executar entre as leituras.
}
```

### Como o código funciona

1. O programa verifica o tempo decorrido usando `millis()`.
2. A cada intervalo de pelo menos dois segundos, solicita uma leitura.
3. Se a leitura falhar e retornar `NaN`, ela é descartada.
4. Uma leitura válida é armazenada na posição indicada por `indice`.
5. O índice avança circularmente.
6. O programa calcula a média das amostras válidas disponíveis.

Durante as primeiras quatro leituras válidas, a quantidade de amostras da média aumenta gradualmente.

A partir da quinta leitura válida, o cálculo utiliza sempre as cinco últimas leituras válidas.

> Se houver falhas de leitura, essas cinco amostras poderão abranger um período maior. O buffer representa as últimas cinco leituras válidas, não uma janela de tempo fixa.

### Por que contar as leituras válidas?

O array começa preenchido com zeros:

```cpp
float temperaturas[numLeituras] = {0};
```

Esses zeros são valores de inicialização, não medições.

Se a primeira leitura for `25 °C`, devemos calcular:

```text
25 / 1 = 25 °C
```

Dividir a soma por cinco nesse momento produziria `5 °C`, pois incluiria quatro posições ainda sem medições.

### Ponteiro como parâmetro de função

Na chamada:

```cpp
calcularMedia(temperaturas, quantidadeValidas);
```

o array é convertido em um ponteiro para seu primeiro elemento.

A função recebe esse ponteiro no parâmetro:

```cpp
const float *leituras
```

O `const` indica que a função não pode modificar os elementos por meio desse ponteiro.

O tamanho é passado separadamente porque o ponteiro, por si só, não informa quantos elementos podem ser acessados.

### Atualização do índice do buffer circular

A expressão:

```cpp
indice = (indice + 1) % numLeituras;
```

faz o índice retornar a zero depois da última posição.

| Leitura válida | Índice de escrita | Índice + 1 | Resto da divisão por 5 | Próximo índice |
|---|---|---|---|---|
| 1 | 0 | 1 | 1 | 1 |
| 2 | 1 | 2 | 2 | 2 |
| 3 | 2 | 3 | 3 | 3 |
| 4 | 3 | 4 | 4 | 4 |
| 5 | 4 | 5 | 0 | 0 |
| 6 | 0 | 1 | 1 | 1 |

Essa operação organiza a escrita circular. O cálculo da média acontece na função `calcularMedia()`.

### Exemplo de cálculo da média

Considere as leituras válidas `25`, `26`, `24`, `25`, `30` e `20 °C`.

| Leitura | Valor recebido | Conteúdo do buffer após a escrita | Amostras utilizadas | Média |
|---|---|---|---|---|
| 1 | 25 | `[25, 0, 0, 0, 0]` | 1 | 25,00 |
| 2 | 26 | `[25, 26, 0, 0, 0]` | 2 | 25,50 |
| 3 | 24 | `[25, 26, 24, 0, 0]` | 3 | 25,00 |
| 4 | 25 | `[25, 26, 24, 25, 0]` | 4 | 25,00 |
| 5 | 30 | `[25, 26, 24, 25, 30]` | 5 | 26,00 |
| 6 | 20 | `[20, 26, 24, 25, 30]` | 5 | 25,00 |

Na sexta leitura, o valor `20` substitui o valor `25` da primeira leitura.

A ordem física do buffer deixa de ser a ordem cronológica das amostras, mas isso não altera a média, pois a soma independe dessa ordem.

### Agendamento com `millis()`

O código evita uma espera com `delay(2000)`, permitindo executar outras tarefas entre as leituras.

Entretanto, a leitura do DHT e a transmissão serial ainda consomem tempo. Portanto, isso não significa que todas as operações sejam instantâneas ou inteiramente não bloqueantes.

---

## 5. GPIO e ligação do botão

GPIO significa *General Purpose Input/Output*: entrada e saída de uso geral.

Um GPIO pode ser configurado para ler um sinal digital ou controlar uma saída, respeitando as capacidades e restrições do pino.

Antes de conectar componentes:

- Identifique o modelo da placa e consulte seu pinout.
- Diferencie o número do GPIO da posição física no conector.
- Verifique se o pino permite a função desejada.
- Confira restrições relacionadas à inicialização e a periféricos da placa.
- Respeite os níveis elétricos permitidos.
- Para ligar um LED externo, utilize resistor limitador de corrente dimensionado para o circuito.

### Montagem de referência do botão

Nesta prática, utilizaremos o **GPIO 27** da placa de referência.

| Terminal do botão | Ligação |
|---|---|
| Primeiro terminal | GPIO 27 |
| Segundo terminal | GND |

O GPIO será configurado como:

```cpp
pinMode(27, INPUT_PULLUP);
```

O pull-up interno mantém a entrada em nível alto quando o botão está solto.

| Estado do botão | Leitura esperada |
|---|---|
| Solto | `HIGH` |
| Pressionado | `LOW` |

> Em botões tácteis com quatro terminais, alguns terminais já são interligados internamente. Confira os pares com um multímetro para que pressionar o botão realmente conecte o GPIO ao GND.

---

## 6. O que é bounce?

Quando um botão mecânico é pressionado ou solto, seus contatos podem abrir e fechar várias vezes em um curto intervalo antes de estabilizar.

Esse fenômeno é chamado de **repique dos contatos**, ou *bounce*.

Sem tratamento, um único acionamento pode ser interpretado como várias mudanças de estado.

### Debounce

Debounce é o tratamento utilizado para evitar que essas oscilações sejam interpretadas como acionamentos independentes.

Ele pode ser implementado:

- **Por software:** aceitando uma mudança somente depois que a leitura permanecer estável por um intervalo definido.
- **Por hardware:** utilizando um circuito adequado para condicionar o sinal.

O repique mecânico e o ruído elétrico são fenômenos diferentes, embora ambos possam produzir alterações indesejadas na leitura.

Adicionar um capacitor sem analisar o circuito não garante uma solução adequada. Em um filtro RC, o comportamento depende da resistência, da capacitância e dos limiares da entrada. Nesta prática, utilizaremos debounce por software.

---

## 7. Prática — Seleção de menu com debounce

O programa a seguir apresenta um menu no monitor serial e avança uma opção a cada pressão válida do botão.

Essa etapa permite verificar o comportamento da entrada antes de integrar o código ao display do projeto.

### Código

```cpp
const int pinoBotao = 27;

const unsigned long intervaloDebounce = 30;
unsigned long instanteUltimaMudanca = 0;

int ultimaLeitura = HIGH;
int estadoEstavel = HIGH;

const char *const opcoes[] = {
    "Temperatura",
    "Historico",
    "Configuracoes"
};

const int quantidadeOpcoes =
    sizeof(opcoes) / sizeof(opcoes[0]);

int opcaoSelecionada = 0;

void mostrarOpcao() {
    Serial.print("Opcao selecionada: ");
    Serial.println(opcoes[opcaoSelecionada]);
}

void setup() {
    Serial.begin(115200);

    pinMode(pinoBotao, INPUT_PULLUP);

    ultimaLeitura = digitalRead(pinoBotao);
    estadoEstavel = ultimaLeitura;

    mostrarOpcao();
}

void loop() {
    unsigned long tempoAtual = millis();
    int leituraAtual = digitalRead(pinoBotao);

    // Reinicia a contagem quando a leitura muda.
    if (leituraAtual != ultimaLeitura) {
        instanteUltimaMudanca = tempoAtual;
        ultimaLeitura = leituraAtual;
    }

    // Aceita a mudanca depois de um periodo estavel.
    if (tempoAtual - instanteUltimaMudanca >= intervaloDebounce) {
        if (leituraAtual != estadoEstavel) {
            estadoEstavel = leituraAtual;

            // Com INPUT_PULLUP, LOW representa uma pressao.
            if (estadoEstavel == LOW) {
                opcaoSelecionada =
                    (opcaoSelecionada + 1) % quantidadeOpcoes;

                mostrarOpcao();
            }
        }
    }

    // Outras tarefas podem executar aqui.
}
```

### Como o debounce funciona

Sempre que a leitura muda, o programa reinicia a contagem do tempo de estabilidade.

A alteração só é aceita quando a leitura permanece sem mudanças por pelo menos `30 ms`.

Esse intervalo é um valor inicial para a prática. Seu ajuste deve considerar o botão utilizado e os resultados dos testes.

O menu avança apenas quando o estado estável passa para `LOW`, indicando uma nova pressão.

Manter o botão pressionado não provoca novos avanços. Para selecionar outra opção, é necessário soltar e pressionar novamente.

### Relação com ponteiros

Na declaração:

```cpp
const char *const opcoes[] = {
    "Temperatura",
    "Historico",
    "Configuracoes"
};
```

`opcoes` é um array de ponteiros constantes para caracteres que não podem ser modificados por esses ponteiros.

Cada elemento aponta para o início de uma string correspondente a uma opção do menu.

### Integração com o display

Após validar o botão no monitor serial, adapte a função:

```cpp
void mostrarOpcao()
```

para desenhar a opção selecionada no display.

A implementação depende do modelo do display e da biblioteca utilizada. A lógica de seleção permanece baseada em:

```cpp
opcoes[opcaoSelecionada]
```

A atualização do display deve acontecer quando a seleção mudar, evitando redesenhos desnecessários a cada passagem pelo `loop()`.

---

## 8. Testes da atividade

### Ponteiros e arrays

1. Execute o exemplo 4.
2. Compare `pt` com `&xyz[2]`.
3. Explique por que `&pt` representa outra informação.
4. Confirme que alterar `*pt` também altera `xyz[2]`.
5. Compare o tamanho de um elemento com o tamanho total do array usando `sizeof`.

### Média das temperaturas

1. Observe a quantidade de amostras indicada nas primeiras leituras.
2. Confirme que ela aumenta até cinco.
3. Verifique que novas leituras substituem as anteriores.
4. Explique por que os zeros iniciais não devem participar do cálculo.
5. Verifique se uma falha de leitura gera uma mensagem e não aumenta a quantidade de amostras válidas.

### Botão e menu

1. Pressione o botão uma vez: o menu deve avançar uma opção.
2. Mantenha o botão pressionado: o menu deve permanecer na mesma opção.
3. Solte o botão: o menu não deve avançar.
4. Pressione novamente: deve ocorrer um novo avanço.
5. Confirme que, após a última opção, o menu retorna à primeira.

Pressões muito curtas, que não permaneçam estáveis durante o intervalo de debounce, podem ser descartadas pelo algoritmo.

---

## 9. Questões para verificação da aprendizagem

1. Qual é a diferença entre o valor de uma variável e seu endereço?
2. O que representam `pt`, `&pt` e `*pt`?
3. Por que um array não é a mesma coisa que um ponteiro?
4. O que acontece ao somar `1` a um ponteiro para um elemento de um array?
5. Por que o tamanho do array é passado separadamente para `calcularMedia()`?
6. Qual é a função do operador `%` na atualização do índice?
7. Por que uma leitura inválida do sensor deve ser descartada?
8. Qual é a diferença entre buffer circular e média móvel?
9. Por que o botão pressionado é lido como `LOW` nesta montagem?
10. Como o código diferencia uma nova pressão de um botão mantido pressionado?

---

## Referências técnicas

- [Documentação de GPIO do Arduino-ESP32](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/gpio.html)
- [Biblioteca DHT da Adafruit](https://github.com/adafruit/DHT-sensor-library)
- [Conversão de array para ponteiro em C++](https://eel.is/c++draft/conv.array)
- [Aritmética de ponteiros em C++](https://eel.is/c++draft/expr.add)
