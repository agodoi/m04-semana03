# Atendimento do Professor

## Terças e quintas. Professor de horário parcial. Nesses 2 dias, podem contar comigo!

# Semana 03 - Conceitos de Ponteiros e Montagem do Botão Antibounce

Entendimento básico da arquitetura de um microcontrolador para dominar Pinout (uso incorreto de pino para led/botão, capacitor debounce, filtragem de ruído), conceitos de ponteiros e memória para programação do microcontrolador ESP32


# Impactos no seu Projeto

* Entenderá a alocação de memória no seu microcontrolador
* Entenderá as funções dos pinos do ESP32.  
* Saberá como resolver o problema do uso de botão para selecionar menu de opções usando o display.

## Ponteiros
- **Ponteiros** são variáveis que armazenam o endereço de memória de outra variável.
- Exemplo de declaração de ponteiro:
  ```cpp
  int x = 9;
  int *pt = &x;
  ```
- **&** é usado para obter o endereço de uma variável.

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

## Funções dos Pinos do ESP32

## Prática - Montagem do Capacitor Antibounce (definir a dinâmica)
