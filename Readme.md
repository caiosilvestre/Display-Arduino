# Sensor de Temperatura com Display
## Projeto Utilizando arduino com Linguagem AVR

[![N|Arduino](https://cdn.arduino.cc/homepage/static/media/arduino-UNO.bcc69bde.png)](https://www.arduino.cc/)


## Componentes:
- 1x Display de 4 segmentos.
- 8x Resistores.
- 1x Botão
- Barramento de pinos para acoplamento no arduino.
- 1x LM35.
- 1x Arduino uno.


## Objetivo do Projeto

- Ler o sinal do LM35 
- Converter o sinal em uma escalas de temperaturas
- Utilizar escalas de Temperatura Celsius, Fahrenheit e Kelvin
- Selecionar a escala que será exibida
- Exibir temperatura em tempo real


## Código

Programa requer [arduino.cc](https://www.arduino.cc/).

## Definir ações para as Portas

```sh
#define acende(Y,bit_x)(Y|=(1<<bit_x)) //ativa o bit x da variável Y (coloca em 1)
#define apaga(Y,bit_x)(Y&=~(1<<bit_x)) //limpa o bit x da variável Y (coloca em 0)
#define cpl_bit(Y,bit_x)(Y^=(1<<bit_x)) //troca o estado do bit x da variável Y
#define tst_bit(Y,bit_x)(Y&(1<<bit_x)) //testa o bit x da variável Y (retorna 0 ou 1)
#define BOT PB5
```

## Atribuir valores as variáveis

```sh
const int pinoPOT = A0;
int time = 5, escala = 10, pres = 0;
char  est = 'A';
const int LM35 = A0; // Define o pino que lera a saída do LM35
float temp = 0; // Variável que armazenará a temperatura medida
float VALOR  = 0;
```
## Definir as portas e os valores inicializados
```sh
void setup() {
  int i, s;
  DDRB =  0B00011111;
  DDRD =  0B11111100;
  PORTB = 0B11000001;
  PORTD = 0B11111100;
}
```
## Loop do programa
```sh
    void loop() {
  sete_seg((int)estado_botao());
}
```
## Digitos
- Criada uma matriz com 7 colunas representando os segmentos.
- 1 == HIGH , 0 == LOW.
- Matriz possue 13 linhas, representam um valor a ser exibido no display.
```sh
byte digits[13][7] = {
  { 1, 1, 1, 1, 1, 1, 0 }, // == Digito 0
  { 0, 1, 1, 0, 0, 0, 0 }, // == Digito 1
  { 1, 1, 0, 1, 1, 0, 1 }, // == Digito 2
  { 1, 1, 1, 1, 0, 0, 1 }, // == Digito 3
  { 0, 1, 1, 0, 0, 1, 1 }, // == Digito 4
  { 1, 0, 1, 1, 0, 1, 1 }, // == Digito 5
  { 1, 0, 1, 1, 1, 1, 1 }, // == Digito 6
  { 1, 1, 1, 0, 0, 0, 0 }, // == Digito 7
  { 1, 1, 1, 1, 1, 1, 1 }, // == Digito 8
  { 1, 1, 1, 0, 0, 1, 1 }, // == Digito 9
  { 1, 0, 0, 1, 1, 1, 0 }, // == Letra C
  { 1, 0, 0, 0, 1, 1, 1 }, // == Letra F
  { 1, 1, 1, 0, 1, 1, 1 }, // == Letra A
};  // 
```
## FUNÇÃO EXIBIÇÃO NO DISPLAY 
- Divide o numero inteiro em centena , dezena ,unidade.
- Assim podendo ser exibido de forma separada no display.
```sh
void sete_seg(int num) {
  int cem = (num % 1000) / 100;             /*Parte responsavel por transformar 1 numero em até 3 numeros, centena, Dezena, Unidade*/
  int dez = ((num % 1000) % 100) / 10;
  int uni = (((num % 1000) % 100) % 10);
  escolha_dig(cem);         /*Parte responsavel por determinar em que digito o valor será exibido*/
```
### Parte responsável atribuir cada valor a sua posição no display
- Chamando a função escolha_pos, responsavel por ativar a posição do display.
- Depois da um delay, em seguida chama outra função, responsável transformar o int em portas para display de 7  seguimentos.
```sh
  escolha_pos(0);
  _delay_ms(time);
  escolha_dig(dez);
  escolha_pos(1);
  _delay_ms(time);
  escolha_dig(uni);
  escolha_pos(2);
  _delay_ms(time);
  escolha_dig(escala);
  escolha_pos(3);
  _delay_ms(time);
  escolha_pos(4);
}
```
## Função responsável por selecionar os digitos
- Deixa apenas 1 digito ativo por vez
- Evitando assim exibir valores indesejáveis
```sh
void escolha_pos(int pos) {
  if (pos == 0)
  {
    apaga (PORTB, 1);
    acende(PORTB, 2);
    acende(PORTB, 3);
    acende(PORTB, 4);
  }
  if (pos == 1)
  {
    apaga(PORTB, 2);
    acende(PORTB, 1);
    acende(PORTB, 3);
    acende(PORTB, 4);
  }
  if (pos == 2)
  {
    apaga(PORTB, 3);
    acende(PORTB, 1);
    acende(PORTB, 2);
    acende(PORTB, 4);
  }
  if (pos == 3)
  {
    apaga(PORTB, 4);
    acende(PORTB, 2);
    acende(PORTB, 3);
    acende(PORTB, 1);
  }
}
```
## Traduz o valor int em sinal para o display
```sh
void escolha_dig(int num) {
  int i, j;
  for (i = 0; i < 7; i++)
  {
    if (i == 6 && digits[num][6] == 1)
      acende(PORTB, 0);
    else if (i == 6 && digits[num][6] == 0)
      apaga(PORTB, 0);
    else if (digits[num][i] == 1)
      acende(PORTD, i + 2);
    else
      apaga(PORTD, i + 2);
  }
}
```
## Convert o sinal do lm 35
```sh
float estado_botao()
{
  temp = analogRead(LM35); //RECEBENDO O VALOR INTEIRO
  temp = ((float)temp); // CONVERÇÃO PARA FLOAT
  temp = (temp * 5.0)/1023.0; // CONVERSAO PARA SINAL DE TENSAO
  temp = temp *100; // RESOLVENDO O VALOR
```
## Código para o botão conseguir selecionar a escala desejada
```sh
  switch (est) {
    case 'A':
      if (tst_bit(PINB, BOT))
      {
        pres = 1;
      }
      if (pres == 1 && !tst_bit(PINB, BOT)) {
        est = 'B';//  cesius
        escala = 11;
        pres = 0;
      }
      temp = temp;
      break;
    case 'B':
      if (tst_bit(PINB, BOT))
      {
        pres = 1;
      }
      temp = (temp * 9) / 5 + 32; //  F
      if (pres == 1 && !tst_bit(PINB, BOT)) {
        est = 'C';//  cesius
        escala = 12;
        pres = 0;
      }
      break;
    case 'C':
      if (tst_bit(PINB, BOT))
      {
        pres = 1;

      }
      temp = temp + 273;
      if (pres == 1 && !tst_bit(PINB, BOT)) {
        est = 'A';//  cesius
        escala = 10;
        pres = 0;
      }
      // K
      break;
    default:
      est = 'A';
      break;
  }
  return temp;
}
```
**Free Code!**


