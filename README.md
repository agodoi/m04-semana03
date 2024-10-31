# Atendimento do Professor

## Terças e quintas. Professor de horário parcial. Nesses 2 dias, podem contar comigo!

# Semana 03 - Conceitos de Ponteiros e Montagem do Botão Antibounce

Entendimento básico da arquitetura de um microcontrolador para dominar Pinout (uso incorreto de pino para led/botão, capacitor debounce, filtragem de ruído), conceitos de ponteiros nos vetores e memória para programação do microcontrolador ESP32


# Impactos no seu Projeto

* Entenderá a importância de vetores (arrays)
* Entenderá a alocação de memória no seu microcontrolador
* Entenderá as funções dos pinos do ESP32.  
* Saberá como resolver o problema do uso de botão para selecionar menu de opções usando o display.

---

## Ponteiros
- **Ponteiros** são variáveis que armazenam o endereço de memória de uma outra variável. Essa variável geralmente é um array ```int v[10] = {5, 10, 15, 3, 10, 76, 5, 13, 33, 45};```
- Exemplo de declaração de ponteiro:
  ```cpp
  int x = 9;
  int *pt = &x;
  ```
- **&** é usado para obter o endereço de uma variável.

### VEJA ESSA [TABELA EXCEL](https://docs.google.com/spreadsheets/d/1zgsKqh5UbhyGN3BkQBueWcM0Ubp_ikUC9deWb7eIvBs/edit?usp=sharing)

### Atenção:
- Manipulação direta de memória pode ser perigosa, devendo ser feita com cautela para evitar falhas e instabilidade do sistema.

---

## Práticas com Ponteiros

### Exemplo 1
```cpp
char x;

void setup(){
  Serial.begin(9600);
  Serial.print("Endereco x: ");
  Serial.println((int)&x);
}

void loop() {}
```

### Exemplo 2
```cpp
char x, y, z;

void setup(){
  Serial.begin(9600);
  Serial.print("Endereco x: ");
  Serial.println((int)&x);
  Serial.print("Endereco y: ");
  Serial.println((int)&y);
  Serial.print("Endereco z: ");
  Serial.println((int)&z);
}

void loop() {}
```

### Exemplo 3
```cpp
char x, y, z;
char xyz[3] = {'x', 'y', 'z'};

void setup(){
  Serial.begin(9600);
  Serial.print("Endereco xyz: ");
  Serial.println((int)&xyz);
  Serial.print("Endereco x: ");
  Serial.println((int)&x);
  Serial.print("Endereco y: ");
  Serial.println((int)&y);
  Serial.print("Endereco z: ");
  Serial.println((int)&z);
}

void loop() {}
```

### Exemplo 4
```cpp
int x = 1000, y = 2000, z = 3000;
int xyz[3] = {222, 333, 444};
int *pt = &xyz[2];

void setup(){
  Serial.begin(9600);
  Serial.print("Endereco pt: ");
  Serial.println((int)&pt);
  Serial.print("Valor de pt: ");
  Serial.println((int)pt);
  Serial.print("Valor apontado por pt: ");
  Serial.println(*pt);
}

void loop() {}
```

---

### Exemplo 5

```
#include <DHT.h>

// Definindo o pino do sensor e o tipo (DHT22)
#define DHTPIN 4
#define DHTTYPE DHT22

// Instanciando o sensor DHT
DHT dht(DHTPIN, DHTTYPE);

// Variáveis para controle de tempo e leituras
const unsigned long intervaloLeitura = 2000; // 2 segundos
unsigned long ultimoTempoLeitura = 0;
int indice = 0;
const int numLeituras = 5;
float temperaturas[numLeituras] = {0};

// Função para calcular a média das leituras de temperatura
float calcularMedia(float *leituras, int tamanho) {
    float soma = 0.0;
    for (int i = 0; i < tamanho; i++) {
        soma += *(leituras + i);  // Usando ponteiro para acessar os valores
    }
    return soma / tamanho;
}

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void loop() {
    unsigned long tempoAtual = millis();

    // Verifica se o intervalo de 2 segundos passou
    if (tempoAtual - ultimoTempoLeitura >= intervaloLeitura) {
        ultimoTempoLeitura = tempoAtual;  // Atualiza o último tempo de leitura

        // Lê a temperatura e armazena no array usando o índice atual
        temperaturas[indice] = dht.readTemperature();

        // Avança o índice, e reseta para 0 se chegar ao número de leituras
        indice = (indice + 1) % numLeituras; //Essa técnica é uma média móvel. Veja a tabela abaixo

        // Calcula a média após preencher o array
        float media = calcularMedia(temperaturas, numLeituras);

        // Exibe a média no monitor serial
        Serial.print("Média de Temperatura: ");
        Serial.println(media);
    }

    // Outras tarefas podem ser executadas aqui sem interrupção
}
```

### Exemplo Prático de Média Móvel

Imagine que numLeituras é 5, então o array **temperaturas** tem 5 posições (índices de 0 a 4). 
Cada vez que adicionamos uma nova leitura, queremos armazená-la no próximo índice do array:

|Iteração|	indice (antes)	| indice + 1	| (indice + 1) % numLeituras	| indice (atualizado) |
|-|-|-|-|-|
|1	|0	|1	|1	|1|
|2	|1	|2	|2	|2|
|3	|2	|3	|3	|3|
|4	|3	|4	|4	|4|
|5	|4	|5	|0	|0|
|6	|0	|1	|1	|1 |


---

## Prática - Montagem do Capacitor Antibounce (definir a dinâmica)
