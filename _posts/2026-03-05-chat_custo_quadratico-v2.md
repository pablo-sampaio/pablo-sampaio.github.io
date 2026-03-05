---
layout: post
title: O Custo Quadrático em Aplicações de Chat
subtitle: Um alerta para quem desenvolve chats com LLMs
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [test]
comments: true
mathjax: true
author: Pablo Sampaio
---

{: .box-warning}
Você está criando aplicações com interfaces conversacionais? Sabia que o custo pode crescer **quadraticamente** com a quantidade de mensagens da conversa?

Essa é uma característica pouco comentada em discussões sobre aplicações com LLMs, apesar de ter implicações importantes.

Neste texto eu explico **qual é esse** custo e **porque** ele existe, apresentando a matemática básica do fenômeno.

---

## 1 - Caracterizando o Problema

Vamos considerar que um chat em *interações*, onde cada interação é um **par**  

`(requisição do usuário → resposta do assistente)`.

Em alguns casos, uma mesma situação pode ser resolvida com $N=3$ interações ou com o **dobro**, que é $N=6$ interações! (Ou mais.)

O que afirmo aqui é o seguinte: *o custo não vai crescer nesta mesma proporção*! (Não é linear!)

Dobrando o tamanho da conversa, você terá algo em torno do **quadráduplo** do valor total! 

> ***De modo geral, chats $N$ interações terão um custo com algum fator $N^2$.***

Para entender o porquê, vamos ver como funciona um chat usando modelos de linguagem.

## 2 - Chat com Modelos Transformers (LLMs e similares)

Aqui, estamos tratando de *aplicações de conversação* baseadas em redes neurais *Transformers Autorregressivas*. Esta é a técnica dos modelos mais famosos atualmente, sejam grandes (como os GPTs, Geminis e Claudes) ou pequenos (como os Gemmas e alguns LLamas e Qwens).

Simplificadamente, estes modelos recebem um *texto de entrada* e retornam um *texto de saída*. Cada texto desse é subdividido em *tokens*, que você pode assumir que são "palavras". (Na verdade, pode ser partes de palavras, pontuações e símbolos).

Por isso, as aplicações de conversação (chats) vão repetir esse loop:

1. O usuário envia uma nova mensagem de requisição.
1. A aplicação coleta o **histórico anterior** da conversa e adiciona a nova requisição ao final (usando tokens "separadores").
1. Esse histórico é enviado ao modelo como "texto de entrada".
1. O modelo gera a próxima resposta (do assistente de conversa) como *texto de saída*. 
1. Essa resposta é acrescentada ao final do histórico. (E exibida ao usuário, geralmente).

**Esse processo se repete, com um histórico cada vez maior!**

Agora, junte essas duas informações:
- uma entrada cada vez maior enviada ao modelo
- modelos cobram (em dólar!) valores que dependem do tamanho da entrada.

Qual a consequência delas? 

**Um custo quadrático**!

Vamos a uma explicação matemática.


## 3 - Modelo Matemático Simplificado

Para modelar matematicamente, vamos assumir algumas simplificações:

- Cada mensagem (do usuário ou do assistente) possui o mesmo tamanho:  
$m \text{ tokens}$.

- Considere que o custo que você paga ao *model provider* (como a OpenAI) é igual **tamanho da entrada** enviada ao modelo, em centavos. Ou seja: 
   - Se você enviar 10 tokens de entrada, o modelo cobra \$ 0,10.

- Não vamos considerar cobrança por:
   - Token de saída (Calma! É uma simplificação!)
   - Tokens que separam as mensagens.

Nas próximas seções:
- Vamos descobrir quantos tokens são usados e cobrados a cada turno de interação.
- Vamos usar isso para definir o custo total de uma conversa.

## 3.1 - Quantos tokens são cobrados por interação?

Vamos definir uma função (/T(n)/) que dá a quantidade de **tokens de entrada** necessária somente para gerar a **$n$-ésima resposta** do assistente. 

Na nossa simplificação, essa função também representa o **custo de gerar uma resposta do assistente** na interação $n$.

Vamos propor uma fórmula para $T(n). Para isso, vamos examinar os valores dessa função à medida que a conversa avança.

### Primeira interação

Lembre-se que a "entrada" enviada para o modelo é o histórico de mensagens, acrescido da mais recente requisição do usuário.

Na primeira interação, o histórico passado é vazio. Assim, será enviada ao modelo apenas 1 mensagem:
  - a primeira requisição do usuário

Essa mensagem tem $m$ tokens (tamanho padrão que assumimos), logo a quantidade de tokens enviados como entrada ao modelo é:

$$
T(1) = 1 \times m
$$

> *Lembrando: A partir dessa quantidade de tokens é que o modelo irá gerar a primeira resposta do assistente.*

---

### Segunda interação

Agora, o histórico terá 2 mensagens passadas e 1 nova:
- requisição 1 do usuário 
- resposta 1 do assistente
- requisição 2 (nova) do usuário

O total de tokens enviados será:

$$
T(2) = 3 \times m
$$

---

### Terceira interação

O histórico a ser enviado conterá cinco mensagens:
- requisição 1 (do usuário)
- resposta 1 (do assistente)
- requisição 2 
- resposta 2
- requisição 3 (nova)

Logo, a quantidade de tokens será:

$$
T(3) = 5 \times m
$$

---

### Quarta interação

(Tente entender sozinho. Pense na quantidade de mensagens passadas e na nova requisição.)

$$
T(4) = 7 \times m
$$

---

### Fórmula geral

Note que, à medida que aumentamos o $n$ da interação, o fator que multiplica $m$ assume valores sucessivos da sequência dos *ímpares positivos*.

Para abreviar a discusão, o padrão geral é dado por:

$$
T(n) = (2n - 1) \times m
$$

Ou seja, **o custo de gerar cada resposta cresce linearmente** com o tamanho da conversa até ali.

> Ai você me diz: "Mas espera! Não era para crescer de forma **quadrática**??"

Calma! Aqui, medimos o custo de gerar apenas 1 resposta!

Vamos analisar o custo total do chat a seguir.

---

## 3.2 - O Ponto Crítico: o Custo Acumulado do Chat

Aqui, queremos quantificar o custo $C(N)$ de uma conversa, como uma função da quantidade de interações $N$ realizadas.

Obivamente, uma conversa (chat) om $N$ interações precisará acionar o modelo $N$ vezes, para gerar as $N$ respostas do assistente.

O custo de cada resposta é dado pelo $T(n)$ que discutimos antes. 

Assim, por definição, o custo total da conversa $C(N)$ é dado pelo os vários valores de $T(n)$ de 1 a N, assim:

$$
C(N) = T(1) + T(2) + T(3) + \dots + T(N)
$$

Substituindo pela fórmula de $T(N)$, isso equivale a:

$$
C(N) = 1m + 3m + 5m + \dots + (2N-1)m
$$

Fatorando por $m$ (à direita):

$$
C(N) = (1 + 3 + 5 + \dots + (2N-1)) \times m
$$

A soma dos **N primeiros números ímpares** é bem conhecida na Matemática, e vale $N^2$. 

Logo, podemos remover o trecho entre parênteses assim:

$$
C(N) = N^2  \times m
$$


Assim, vemos que o custo total $C(N)$ tem sim um fator quadrático!

(Que fique claro: trata-se do custo para usar o modelo para gerar as respostas ao longo de toda a conversa).

Resumimos essa conclusão assim:

**o custo total de um chat cresce quadraticamente com o número de interações.**

---

# 4. O que isso significa na prática?

Essa propriedade tem algumas consequências importantes que podem não ser intuitivas. 

Cito algumas apenas para exemplificar, sem esgotar o assunto:

## 4.1 - Custo de conversas longas cresce rápido demais

Com a relação quadrática encontrar entre o tamanho da conversa $N$ e o custo $C(N)$, os custos entre conversas terão relações assim:
- Se uma conversa dobrar de tamanho (ex.: de 3 para 6 interações), o seu custo quadruplica ($\times 4$).
- Se ela triplicar de tamanho (ex.: de 3 para 9 interações), o seu custo é multiplicado por nove ($\times 9$).
- Se ela quadruplicar de tamanho (ex.: de 3 para 12 interações), o seu custo é multiplicado por dezesseis ($\times 16$).

Enfim, **chats longos são estruturalmente caros**. 

> Tem certeza que quer deixar o seu usuário conversar livremente?

---

## 4.2. Engenharia de Contexto passa a ser Essencial

Ok, você até pode deixar seu usuário conversar livremente (ou quase isso). Mas precisa definir uma estratégia para diminuir o impacto.

Algumas técnicas simples para lidar com isso:
- truncamento de histórico
- sumarização de conversas
- salvar parte da conversa em memória externa, para recuperar quando necessário
- iniciar um novo contexto, quando a requisição for nova (não for continuação do assunto anterior)

> Essas técnicas fazem parte da chamada **Engenharia de Contexto** (qua trata de como manter o contexto mais relevante possível, com o menor tamanho possível).
> O que é chamado de "contexto", aqui, é a apenas "entrada" do modelo. Que é simplesmente o "histórico", no caso tratado aqui.

---

# 5. Conclusão

Na prática, o custo real pode diferir das fórmulas acima por alguns motivos:
- o valor por token varia conforme o modelo
- é comum ter diferentes valores por token de entrada e por token de saída
- mecanismos de **prompt caching** oferecidos por alguns provedores pode baratear o histórico passado

Mesmo levando essas características em consideração, a **tendência quadrática continua válida**.

Quando pensamos em escala — milhares ou milhões de conversas — entender a matemática do contexto se torna fundamental.

E, aqui, quis apenas mostrar a importância do tema. Você precisa estar atento a isso ao criar uma aplicação conversacional.

---