#include <LiquidCrystal.h>
#include <Keypad.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

const byte LINHAS = 4;
const byte COLUNAS = 4;

char teclas[LINHAS][COLUNAS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte linhaTeclada[LINHAS] = {9, 8, 7, 6};
byte colunaTeclada[COLUNAS] = {A0, A1, A2, A3};

Keypad teclado = Keypad(makeKeymap(teclas), linhaTeclada, colunaTeclada, LINHAS, COLUNAS);

const int ledVerde = 10;
const int ledVermelho = 13;
const int buzzer = A5;

int sequencia[8];
int tentativas[8];
int rodada = 0;
int numeroSorteado = 0;
int nivelDificuldade = 1; // 1 a 5

void setup() {
  lcd.begin(16, 2);
  pinMode(ledVerde, OUTPUT);
  pinMode(ledVermelho, OUTPUT);
  pinMode(buzzer, OUTPUT);

  lcd.print("Jogo da Memoria");
  delay(2000);
  lcd.clear();
  selecionarDificuldade();
  iniciarNovoJogo();
}

void loop() {
  numeroSorteado = 0;

  if (rodada < nivelDificuldade) {
    numeroSorteado = random(0, 10);
    sequencia[rodada] = numeroSorteado;
    mostrarSequencia();

    if (aguardarEntrada()) {
      rodada++;
      digitalWrite(ledVerde, HIGH);
      delay(500);
      digitalWrite(ledVerde, LOW);
    } else {
      digitalWrite(ledVermelho, HIGH);
      tone(buzzer, 1000, 500);
      delay(1000);
      digitalWrite(ledVermelho, LOW);
      iniciarNovoJogo();
    }
  } else {
    lcd.clear();
    digitalWrite(ledVerde, HIGH);
    digitalWrite(ledVermelho, HIGH);
    for (int i = 0; i < 5; i++) {
      tone(buzzer, 1000, 500);
    }
    lcd.print("Voce venceu!!!");
    delay(5000);
    selecionarDificuldade();
    iniciarNovoJogo();
  }
}

void mostrarSequencia() {
  lcd.clear();
  lcd.print("Memorize:");
  lcd.setCursor(0, 1);

  for (int i = 0; i <= rodada; i++) {
    lcd.print(sequencia[i]);
    lcd.print(" ");
  }

  delay(2000);
  lcd.clear();
}

bool aguardarEntrada() {
  lcd.print("Digite:");
  int index = 0;

  while (index <= rodada) {
    char chave = teclado.getKey();

    if (chave != NO_KEY) {
      if (chave >= '0' && chave <= '9') {
        tentativas[index] = chave - '0';
        lcd.setCursor(index, 1);
        lcd.print(chave);
        index++;
      }
    }
  }

  for (int i = 0; i <= rodada; i++) {
    if (tentativas[i] != sequencia[i]) {
      return false;
    }
  }
  return true;
}

void iniciarNovoJogo() {
  rodada = 0;
  lcd.clear();
  lcd.print("Novo Jogo");
  delay(2000);
  lcd.clear();
}

void selecionarDificuldade() {
  lcd.clear();
  lcd.print("Dificuldade:");
  lcd.setCursor(0, 1);
  lcd.print("1-5");

  while (true) {
    char chave = teclado.getKey();
    if (chave != NO_KEY) {
      if (chave >= '1' && chave <= '5') {
        nivelDificuldade = chave - '0'; // Atualiza o nível com a tecla pressionada
        lcd.clear();
        lcd.print("Nivel:");
        lcd.print(nivelDificuldade);
        delay(2000);
        break;
      }
    }
  }
}