# ALS Hook Methor - PTBR

## Creditos: ! FBI
### Github: [Masanb](https://github.com/Masanb)
### Discord: ! FBI#2799

<br>

### Nessa thread eu estarei basicamente explicando um método chamado ALS Hook que se trata de um bug do compilador que permite você fazer "hooks" ou chamadas de funções já existentes em outras partes do seu código, permitindo assim você dividir o seu código em módulos. Normalmente isso é feito com o Y_hooks da biblioteca YSI, porém existem motivos para optar pelo método ALS e irei detalhar melhor durante a thread.
<br>

>## Prólogo

- ### Bem, como descrito acima, fazer hooks de callbacks por exemplo te permite que você divida o seu código em várias partes, ou módulos. Tentar chamar a mesma callback várias vezes sem fazer um hook só te retornará erro avisando que a callback já existe, por isso são necessários métodos hook para isso. No momento é muito usada a include Y_Hooks do Y_Less(criador da include), porém essa include trás problemas como, precisar de toda a biblioteca YSI mesmo que você queira usar apenas o Y_hooks, atrasos na hora de compilar, erros de compilação por conta da dependência da biblioteca e claro, problemas para compilar em certas plataformas.

>## O'que é o método de hook ALS?

- ### Basicamente um bug no compilador, você usa de diretivas de processamento para chamar uma callback por mais de uma vez no código, redirecionar funções nativas do SAMP para funções próprias que você pode implementar na unha e possibilitando fazer coisas que já seriam feitas antes usando o Y_Hooks porém sem a include, claro.

>## Como fazer? 

- ### Ok, vamos por partes pois aqui pode ser um pouco complicado para aqueles que não tem muita familiaridade com algumas diretivas de pré-processamento e por isso irei explicar, para aqueles que lerem um pouco e perceberem que já entendem as diretivas que são usadas, apenas pulem pro próximo tópico se for de sua vontade.

>## Diretivas de pré-processamento

- ### Para você que não sabe o'que são essas diretivas da quais falei, se você já programou algo de verdade, já usou elas sem nem mesmo perceber. As diretivas são em sua grande maioria aquelas linhas que começamos com o prefixo "#". Te lembra algo? algo como isso:
~~~c
#include
#define
~~~
 
- ### Nesse ponto você já deve ter entendido a ideia, o fato é que essas diretivas dizem ao compilador algumas instruções que ele deve fazer durante o processo de compilação, no caso das diretivas acima, são as de inclusão de arquivos, e de definição de constantes, respectivamente. 

### Abaixo estarei listando as diretivas que usamos durante o método de hook e sua funcionalidade.
~~~c
#if // Testa o valor de uma constante(valor definido pela diretiva #define)
#else // Usada igual nas condicionais convencionais porém, para nesse caso, se o #if não seja satisfeito
#endif // Sinaliza o fim de uma condicional feita pelas diretivas
#defined // Indica se uma constante já foi definida( Constante definida pela diretiva '#define')
#undef // Serve para indefinir uma constante ( Constante definida pela diretiva '#define')
#define // Define constantes durante a compilação
~~~

> ## Prática com funções

- ### Agora já temos uma noção sobre as diretivas, então vamos por a mão na massa.
- ### O primeiro passo é entender oque podemos fazer hookando funções e eu te respondo, redirecionar ou "substituir" a função original. No exemplo a seguir iremos fazer com que toda chamada pra função SetPlayerHealth seja redirecionada pra uma função que nós vamos criar. Essa função vai além de setar a vida do jogador, vai realizar um log imaginário que admins imaginários teriam acesso.

~~~c
// Coloque o nome que quiser pra sua função, não há regras e não precisa colocar o 'HOOK_', fiz apenas por estilo mesmo.
HOOK_SetPlayerHealth(playerid, Float:health)
{
    Log("O jogador %s teve sua vida alterada para %.2f pelo servidor.", Player[playerid][pName], health); // Nossa função de Log imaginária junto a uma variável 'Player[playerid][pName]' que contém o nome do jogador, imaginária.
    SetPlayerHealth(playerid, health);
    return 1;
}
// Agora vamos ao que interessa. Hora de substituir as chamadas da função SetPlayerHealth nativa pela nossa.
#if defined _ALS_SetPlayerHealth
  #undef SetPlayerHealth
#else
  #define _ALS_SetPlayerHealth
#endif
#define SetPlayerHealth HOOK_SetPlayerHealth
~~~

- ### E pronto, agora toda vez que você chamar a função SetPlayerHealth, o código executado será o código da nossa função HOOK_SetPlayerHealth, foi bem fácil né? 

>## Prática com Callbacks

- ### Bem, callbacks são mais completinhas e são chamadas constantemente no nosso servidor já que respondem quando um evento acontece. Porém a lógica não foge do que já sabemos até agora, embora o código a seguir pode parecer meio... confuso.

~~~cs
public OnPlayerConnect(playerid)
{
  SendClientMessage(playerid, -1, "Seja bem-vindo ao Venecos Academy!");
  return 1;
}
~~~


- ### Ok, não há nada demais no código acima, correto? então vamos partir pro problema real, imagine que queremos adicionar um código para quando o jogador se conectar, aparecer uma dialog de login e registro PORÉM, em outro lugar do nosso gamemode ou até mesmo em outro arquivo(módulo). Se apenas chamarmos a callback novamente, iremos ter um erro indicando que ela já foi definida, então como faremos? Hookando ela! o ALS é um meio de trabalhar com a lógica de definição de bibliotecas, por isso essas diretivas são executadas antes mesmo das callbacks originais, então se quisermos chamarmos essa callback de outro lugar do código, temos que chamar ela "antes" que seja definida.  
- ### Aqui a lógica já passa a ficar um pouco confusa mas não se esqueça, isso se trata de um bug e por isso não precisa entender a fundo do porque ela é considerada como não definida em certo ponto ou não, é apenas um bug e usamos dele para isso e funciona perfeitamente.

## Vamos ver como poderiamos fazer esse hook da callback em outro arquivo separado:
~~~cs
// vamos supor que estamos no arquivo "../modules/login.pwn"

public OnPlayerConnect(playerid)
{
  ShowPlayerDialog(playerid, 101, DIALOG_STYLE_INPUT, "Login - Venecos Academy", "Digite sua senha para logar.", "Logar", "Sair");
  return 1;
}

#if defined _ALS_OnPlayerConnect
  #undef OnPlayerConnect
#else
  #define _ALS_OnPlayerConnect
#endif
#define OnPlayerConnect HOOK_OnPlayerConnect
#if defined HOOK_OnPlayerConnect
  forward HOOK_OnPlayerConnect(playerid);
#endif
~~~

- ### E prontinho, esse código já vai funcionar para chamar a public OnPlayerConnect quantas vezes vocês quiser e de "onde quiser".  
Tanto nas callback's quanto nas funções, percebam que temos um padrão de usarmos um _ALS_ antecedendo oque queremos fazer hook. É obrigatório que sigam esse padrão e escrevam esse prefixo para funcionar.

- ### Sei que você você pode estar se perguntando de onde surgiu aquele HOOK_OnPlayerConnect, talvez até tenha pensado que eu tenha ficado louca mas não, para isso funcionar devemos definir isso como se fosse outra callback, com outro nome, porém o código que de fato será executado é o da callback logo acima das diretivas com o nome OnPlayerConnect original.

>## Resultado

- ### Ao entrarmos  no nosso servidor imaginário, iremos ter a mensagem que está  sendo enviada na OnPlayerConnect presente  no nosso arquivo principal  do gamemode e também a dialog de login imaginária que programamos para aparecer quando o jogador logar, aqui no nosso arquivo "login.pwn" imaginário.

>## Conclusões finais   
 
- ### Na prática existem sim algumas diferenças e caso tente replicar o'que fizemos aqui hoje e tenha algum erro, basta me chamar que estarei disposta a ajudar.
- ### Também gostaria de dizer que não há problemas em usar o Y_Hooks, esse método é realmente para aqueles que não gostam do atraso pra compilar com o Y_hooks devido a lib YSI, não consegue dividir o gamemode em módulos devido ao Y_hooks não compilar em certas plataformas, caso vá fazer uma include pública e precisa chamar algumas callbacks no seu código e pra isso é quase que inviável usar o Y_Hooks por deixar sua include super pesada, além de N outras situações úteis pra tudo que aprendemos aqui. Use oque achar melhor ;)
