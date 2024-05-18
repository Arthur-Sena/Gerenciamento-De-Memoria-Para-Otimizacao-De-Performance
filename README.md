# Curso .NET: GERENCIAMENTO DE MEMÓRIA PARA OTIMIZAÇÃO DE PERFORMANCE

Nsse curso de .NET:

- Entenda como listas são armazenadas em memória
- Conheça os conceitos de Stack, Heap e Large Object Heap e compreenda onde as informações são armazenadas
- Aprenda o que são e como utilizar structs e records
- Entenda o que são e como utilizar record structs

## Resumo

- Structs são **tipos por valor** (Seção 11.3.1).
- Todos os tipos struct implicitamente herdam da classe **System.ValueType** (Seção 11.3.2).
- Atribuição a uma variável do tipo struct cria uma cópia do valor sendo atribuído (Seção 11.3.3).
- O valor padrão de uma struct é o valor produzido após atribuir todos os tipos valores para seu valor padrão e todos os tipos referência para null (Seção 11.3.4).
- Operações de boxing e unboxing são usadas para converter entre um tipo struct e um objeto (Seção 11.3.5).
- O significado da palavra-chave this é diferente para structs (Seção 11.3.6).
- Não é permitido que declarações de campos de instância para uma struct incluam inicializadores de variáveis (Seção 11.3.7).
- Uma struct não pode declarar um construtor de instância sem parâmetros (Seção 11.3.8).
- Uma struct não pode declarar um destrutor (Seção 11.3.9).

  
*Este resumo é uma tradução da documentação da Microsoft e foi realizada pelo mgibsonbr. Mas ele teve seus motivos para remover a resposta postada. Coloquei aqui porque é uma forma que ajuda os usuários.*

## Detalhamento
### Struct:
- Structs são usadas para criar estruturas de dados cujas instâncias (os objetos) sejam pequenas (no máximo 16 bytes), sejam imutáveis, representem um valor único, ou seja, que não contenha diversas características, e não precise ser encapsulado (boxing) em objetos por referência com frequência. Estas estruturas têm seu conteúdo (seu valor) integral copiado quando precisa transportar de um local para outro. Portanto são tipos por valor (value types).

### Cópia de valores:
O fato de todos os membros serem copiados facilita o seu uso em ambientes concorrentes. Há garantia que uma instância será acessada exclusivamente pela thread que está em execução. Só é preciso ter cuidado com structs que contém referências dentro dela. Os objetos referenciados não estão "protegidos" contra acesso concorrente. Uma struct não é segura em ambiente concorrente só porque é uma struct, depende de como ela foi definida pelo programador. A atomicidade, por exemplo, não é garantida intrinsecamente.

### Herança:
Além disso, **structs não permitem herança**, ainda que uma struct seja sempre derivada implicitamente de uma classe chamada **ValueType** que por sua vez é derivada de outra classe Object como qualquer tipo no CLR.

#### Como assim, uma struct é derivada de uma classe? 
  - *Sim, **ValueType** é uma classe abstrata e Object só contém comportamento e não estado. Não existe nada de estranho nisso, estas classes apenas fazem parte da hierarquia do tipo. Então está fazendo subtipo, mas não subclasse. É como se tivesse herdando apenas de uma interface.*

Não existe nada que obrigue qualquer tipo por valor ser herdado só por tipos por valor, na verdade isto nem é possível. Claro que se existisse estado seria mais complicado, afinal o estado teria que ser armazenado em algum lugar e é esperado que ele seja acessado por uma referência. No caso sem estado a referência não importa. Apenas o comportamento é herdado pelo tipo por valor, ou seja, o tipo por valor pode chamar diretamente um método definido e/ou implementado nestas classes superiores, nada mais que isto, ele não recebe estado.

Herança indica apenas uma forma de relacionamento "é um" entre a base e a derivada e não implica em mudança na forma do armazenamento (por referência ou por valor) do estado que será definido pela estrutura derivada. Obviamente que a derivada não pode ter menos coisas que a base.

### Outras limitações:
Uma struct não pode ter um construtor sem parâmetros (chamado de default) (Agora pode). Na prática existe um construtor padrão, mas o programador não pode defini-lo por conta própria em C# (isto não é verdade para o CLR e outras linguagens que podem usar outra forma. Isto precisa ser entendido quando vai se fazer interoperabilidade com linguagens diferentes do CLR).

Quando um objeto (uma instância) struct é criado, seu valor precisa ser inicializado sempre. A inicialização se dá através dos valores padrão de todos os membros da struct.

Também não possuem destrutores. Uma struct é destruída quando ela sai de escopo, quando ela não é mais necessária como variável local ou quando um outro objeto que contém a struct é destruído.

#### Onde eles são usados?
- *As structs existentes no .NET dão uma ideia de como estes tipos devem definir dados com uma característica única, devem representar um dado único. Exemplos: int, double, bool, Decimal, DateTime.*

O último mostra que uma informação simples pode ter partes, no caso a data e a hora, mas ainda assim é um dado único. Diferente de um calendário que possui diversas informações relacionadas. Calendar é uma classe e não uma struct.

### Class:
Classes são usadas para todos os casos que uma struct pode trazer problemas, principalmente quando não atendem as recomendações acima. E elas são muito mais comuns nas aplicações.

### Evitar cópia:
É comum você ter objetos que possuem várias características e consequentemente, são maiores. Como os objetos são grandes, fazer cópias pode se tornar muito custoso em termos de processamento, pode desperdiçar memória e colocar pressão no garbage collector, por isso esses objetos costumam ser mutáveis, ou seja, eles podem ter suas características alteradas individualmente.

Em objetos mutáveis é normal que apenas a referência é copiada. Neste caso, são tipos por referência (reference types). Nada impede que uma classe ou outro tipo por referência seja definida para trabalhar com objetos imutáveis. A classe String é o maior exemplo disto.

A destruição de um reference type é feita pelo garbage collector.

Seu valor default é null que é um endereço inválido da memória.

### Boxing:
Um tipo por referência, como é o caso de uma classe, pode ser usado para encaixotar um tipo por valor (boxing), criando uma referência para o seu valor. Esta técnica deve ser evitada tanto quanto possível. A criação dos tipos genéricos permitiu que este tipo de operação fosse largamente evitado.

A grosso modo isto é feito:

class BoxedInt {
    int value;
    public BoxedInt(int v) { this.value = v; }
    public int Unbox() { return this.value; }
}
Então o código:

int i = 1;
object obj = i;  
int j = (int)obj;
seria convertido para:

int i = 1;
object obj = new BoxedInt(i);
int j = ((BoxedInt)obj).Unbox();

### Terminologia:
Para entender alguns conceitos apresentados aqui, pode consultar outras respostas sobre o assunto: imutabilidade e Reference types vs value types que são conceitos que se confundem com classes e structs, respectivamente. Os termos não chegam ser intercambiáveis já que uma classe não é a única forma de criar um reference type, existem os delegates e interfaces por exemplo, e além da struct existe o value type enum.

### Abuso de struct:
É comum programadores inexperientes darem preferência para structs achando que estão otimizando alguma coisa, quando, na verdade, o resultado pode ser o contrário. Além disto sem tomar alguns cuidados, seguir as recomendações descritas acima, é comum ter problemas na utilização desses tipos. Só devemos optar por criar uma struct quando existe uma boa razão para isto.

*Note que foram descritas recomendações para o que deve ser uma struct. A própria Microsoft as viola em vários exemplos dentro do .NET. Recomendação não é regra obrigatória. Você pode fugir destas características quando sabe o que está fazendo e tem um bom motivo para isto. Estas características são esperadas pelos programadores quando estão usando uma struct definida, portanto se houver um bom motivo para não segui-las isto precisa estar bem documentado.*

Não é fácil entender as implicações de um tipo por valor. Apesar de poder fazer em uma struct quase tudo o que uma classe pode, deve-se evitar situações complexas que podem não ser tão ortogonais ou que podem ser entendidas equivocadamente. O conceito de cópia do valor é simples para operações simples, mas se torna complexo quando inserido recursos que não se adéquam bem neste tipo de dado, criando armadilhas. Um evento, por exemplo, é indesejável.

Structs são muitos úteis, podem otimizar código e dar a semântica correta, mas elas não são absolutamente necessárias. Java mesmo viveu sem elas até hoje (claro que algumas operações não são trágicas em performance porque a linguagem possui tipos primitivos que no fundo têm a mesma característica de uma struct do C#). Se o programador não consegue entender todas implicações de uma struct é provável que seja melhor ele optar por uma classe. Não é o ideal, mas é o mal menor.

<!-- 
### Característica	Struct |	Class
- Tipo por referência ou valor:	valor	| referência
- Pode ter tipos aninhados (enum, class, etc.): Sim |	Sim
- Pode conter constantes:	Sim |	Sim
- Pode ter campos:	Sim |	Sim
- Campos podem ser inicializados automaticamente:	Não |	Sim
- Pode ter propriedades, indexadores e métodos:	Sim	| Sim
- Pode ter eventos:	Sim	| Sim
- Pode ter membros estáticos:	Sim |	Sim
- Permite herança:	Não (herda de ValueType) é sempre selado | Sim
- Permite implementação de interfaces:	Sim	| Sim
- Pode sobrepor os construtores e operadores:	Sim	| Sim
- Permite definir construtor default:	Não	| Sim
- Pode usar generics:	Sim	| Sim
- Pode ser parcial:	Sim	| Sim
- Implementação do this:	variável com o valor	referência | só leitura
- Uso do operador new:	opcional | obrigatório
-->
<img width="600" alt="image" src="https://github.com/Arthur-Sena/Curso-GERENCIAMENTO-DE-MEMORIA-PARA-OTIMIZA-O-DE-PERFORMANCE/assets/57300757/169b0b52-3053-4302-b5a2-59808f9698fb">
