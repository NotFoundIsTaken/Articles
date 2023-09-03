## Introdução

Rust é estranho. Eu sei, você sabe, o psicopata maluco que diz que JavaScript é a melhor linguagem sabe. Ainda assim, uma das coisas mais esquisitas em Rust é definitivamente a forma que ele gerencia a memória. São uma série de regras sem pé, nem cabeça que quase ninguém entende no inicio e a única utilidade aparente é fazer seu código não compilar. E é isso que a gente vai estudar hoje.

Um breve resumo do que virá:

- **Stack** é usada para armazenar dados de tamanho **conhecidos** durante a compilação,
- **Heap** é usada para armazenar dados de tamanho **desconhecidos**.
- Inteiros como u8, u16, u32, u64 e floats são salvos na Stack (tem tamanho definido na compilação).
- Strings são salvas na Heap (seu tamanho é indefinido até o uso).

## Heap e Stack

Okay, eu sei que você tá com pressa. O clima tá esquisito, a coca retornável anda vindo sem gás, o cachorro cagou todo quintal e seu filho psicótico tá do seu lado esmagando uma banana com a cabeça do boneco do homem aranha. Sua vida é importante demais para programação de baixo nível, mas eu garanto que isso vai ser importante. 

Primeiro, imagina o seguinte: 

~~Você é um nóia e precisa esconder a maconha da policia~~  Você é um pai, de família, com um emprego honesto, e precisa guardar um presente até o aniversário do seu filho. **Sabendo o tamanho** do brinquedo mesmo **antes** de comprar você **arranja um espaço** na garagem. Quando o ~~pm~~ brinquedo chega você o guarda rapidamente onde ninguém possa ver.

Em Rust, há certos valores que tem o tamanho conhecido mesmo antes de serem usados, por exemplo os números inteiros. Você pode definir o tamanho máximo do número mesmo antes de precisar dele, assim fica extremamente rápido guardar ele na memória. Esse espaço reservado para **valores de tamanho conhecido** se chama **Stack**.

Agora, supondo que você estivesse prestes a receber um carregamento de "presentes" de tamanho desconhecido e, logo após receber eles, seu "filho" chegasse em casa. Você então entraria em desespero, olharia para todos os lados pensando onde guardar o pacote e no final, já em prantos pensando na sua castidade retal após derrubar sabonetes em ambientes insalubre, decidiria engolir tudo de uma vez e preservar suas pregas. Rust é mais inteligente que você.

Quando o Rust **não sabe o tamanho** de algo ele também procura um espaço onde possa colocar esse valor. Infelizmente não pode ser no seu estômago, então ele armazena em local chamado **Heap**. 

Ambas, Heap e Stack, são convenções na forma de utilizar a memória disponível. Heap aceita qualquer porcaria, enquanto Stack é fresca mas eficiente. 

## Strings

"Uma string é como uma corrente de contas ou um colar de contas, mas em vez de serem contas reais, são letras, palavras e até mesmo frases! É como juntar letras para fazer uma mensagem especial." - GPT, Chat (2023).

Pra quem é tapado igual eu, string é uma listinha de letra. Se você for bom o suficiente com elas dá até pra fazer palavras. Dito isso, Strings em Rust são uma putaria completa. 

Tecnicamente existem três tipos, mas só dois são relevantes:

- **&str**, conhecido como string slice, é uma referência. Isso quer dizer que não possui um valor real, mas aponta pra algo que já está salvo em algum outro lugar. Geralmente uma String ou um texto escrito direto no código. 
- Já **String** é... uma string. Eu sei, eu sei. O tipo **String** é o que você consideraria uma string comum:  um conjunto de caracteres. É o tipo que você usa pra pegar um input, salvar algum dado e etc.

_Sim, eu sei que você, pessoal normal, não entendeu nada sobre **&str**. Já o crackudo do C provavelmente percebeu que isso é um ponteiro. Sim, referencias em Rust são abstrações de ponteiros, mas não conta pra ninguém._ 🤫

Okay, mas pra que eu preciso saber disso? 

Sobre ponteiros? Não precisa. O importante aqui é o tipo String, pois ela não tem tamanho definido durante o tempo de compilação. Isso quer dizer que o Rust, antes de você escrever algo, não tem como saber o tamanho do que você tá escrevendo. Pode ser o seu nome, mas também pode um dissertação sobre as proporções do orgão fálico de um cavalo. Rust não sabe até que você digite de fato e isso só ocorre quando o programa já está rodando. 

### Números

Rust tem uma porrada de tipo número. Considerando só os inteiros, ou seja, os números "sem vírgula", há pelo menos 10. 

![image](https://github.com/NotFoundIsTaken/Articles/assets/49632633/f63f5194-2430-4654-926b-292c925cba26)

O motivo de existir tantos tipos é por levar em consideração o tamanho antes da compilação. Um número i8, ou seja, inteiro de 8 bit, pode ir de 0 até 255. Enquanto um i128 vai até 340.282.366.920.938.463.463.374.607.431.768.211.456. Essa porcaria é tão grande que eu nem sei falar o nome. 

Okay, mas porque o inteiro tem tamanho definido e a string não? 

Vamos discutir o tamanho de uma string que contém um nome. Por exemplo, meu nome possui 18 letras e dois espaços, o que equivale a aproximadamente 20 bytes, ou seja, 160 bits. Isso é consideravelmente menor do que o número mencionado anteriormente.

Agora, se pensarmos em uma redação sobre o tamanho da trolha de um cavalo, ela pode facilmente ultrapassar os 8000 bits (de acordo com minhas pesquisas). Essa diferença é significativa demais para justificar a criação de diferentes tipos de strings com tamanhos predefinidos. Portanto, é melhor deixar que o tamanho das strings ser definido dinamicamente durante o uso, acomodando assim uma ampla variedade de informações sem desperdiçar espaço em memória.

## Ownership

Depois de muita enrolação descobrimos que tem dois tipos de variáveis: a com tamanho conhecido durante a compilação e a ~~bilonga de cavalo~~ com tamanho de conhecido durante a execução. As regras do Rust giram em torno disso. 

Lembrando que:

- Tempo de compilação é quando você planeja que vai sair de casa, toma banho, passa perfume, chama a novinha.
- Tempo de execução é quando você tá no rolê, bebe demais, passa mal e leva um fora. 

Em Rust, a propriedade (ownership) é o conceito que define as regras de relação entre uma variável e os dados que ela contém. Ela é a mãe chata que fica enchendo o saco sobre qual filho é dono do brinquedo. 

Bora vê um exemplo com u32, um tipo inteiro (tamanho definido na compilação):

```Rust
// caso 1: número inteiro
let x: u32 = 5;
let y: u32 = x;
```

Se você tem experiência com programação, deve poder imaginar o que acontece aqui. A variável `x` recebe o valor 5, e então a variável `y` recebe a variável `x` (que contém o valor 5). No final, `x` e `y` são ambas iguais a 5.

No exemplo seguinte, temos uma situação semelhante, mas com String (tamanho definido na execução):

```Rust
// caso 2: string de tamanho variável
let s1: string = String::from("hello");
let s2: string = s1;
```

A variável `s1` recebe uma string, `s2` recebe `s1` e assim por diante. No final, esperamos que `s1` e `s2` sejam ambas "hello", certo? Bem, não.

O lógica é meio esquisita, então me segue:

- **String** não tem tamanho definido na compilação, por tanto é salva na memória Heap.
- A memória Heap é lenta, então o Rust caga pra suas necessidades de copiar o valor.
- O valor então, ao invés de copiado, é movido diretamente de `s1` para `s2`.
- `s1`, agora vazio, deixa de existir.

Ué, mas "hello" tem 5 letras, tá escrito ali, como que o Rust não sabe o tamanho?

Não saber o tamanho durante a compilação é uma propriedade inerente do tipo string. É como um corno que não aceita o chifre mesmo depois de ser informado. O tipo String foi feito para ser alocado na memória Heap e é isso que vai ser feito. 

## Copy e clone

Antes de prosseguir, é interessante entender que existe uma forma de manter tanto s1 quanto s2, e isso é feito usando o método clone. 

```Rust
fn main() {
  let s1 = String::from("name");
  let s2 = name.clone();

  println!("{}, {}", s1, s2);
}
```

Todo:
- explicar uso em structs
- adicionar mais código e imagens
- reduzir tamanho do texto
- explicar referencias
- explicar borrowing
- explicar sobre string slice
- adicionar recomendações uteis
