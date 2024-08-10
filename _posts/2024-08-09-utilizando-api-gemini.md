---
layout: post
title: Utilizando a API do Google Gemini
subtitle: Use a Mais Recente IA Generativa do Google em Python
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [books, test]
author: Vinicius B. Pessoa
---

Já vimos aqui no blog como escrever programas em Python integrados o **ChatGPT**. 
Neste artigo, veremos como fazer o mesmo, porém integrando com o **Gemini**, da Google.

Apenas lembrando que não vamos usar a interface de usuário do ChatGPT. Todo o acesso será feito
por meio do código de um programa Python, usando uma **API** (Interface de Programação de Aplicações) própria do Gemini.

E o que podemos fazer assim? Podemos criar programas que usam o ChatGPT adaptado ao contexto específico da sua aplicação. Por exemplo:
- Você pode dar, ao ChatGPT, acesso a dados específicos do usuário, como documentos ou atividades agendadas.
- Você pode personalizar o ChatGPT para disparar código Python para realizar cálculos complexos ou outras ações.
- O ChatGPT pode fazer resumos, classificar textos, ou fazer outro tipo de raciocínio em linguagem natural.

Enfim, você pode mesclar a capacidade de linguagem e raciocínio do ChatGPT com a capacidade de programação Python tradicional para criar programas inovadores!

Neste documento, você saberá como acessar e utilizar o **Google Gemini** para esse fim.

# 1 - Configurações Iniciais

## Pre requisitos

1 - Python 3.9 ou superior.

2 - Uma instalação de jupyter para executar o notebook.

3 - Possuir a seguinte biblioteca instalada (google-generativeai)

```bash
pip install google-generativeai
```
O seguinte modulo ira instalar a biblioteca **google-generativeai** caso a mesma  não esteja instalada.

## Obtendo uma chave para sua API

1 - Acesse o seguinte site: [Google API Key.](https://aistudio.google.com/app/apikey?hl=pt-br)

2 - Caso ainda não esteja logado, faça o login utilizando sua conta Google e aceite os termos de serviço.

3 - Clique no botão **Criar chave de API** para criar uma chave para um novo projeto.

4 - Nesse momento será exibido uma mensagem com os termos de serviço, basta clicar em **OK** para prosseguir.

5 - Uma caixa contendo a opção de **Criar uma chave de API em um novo projeto**, Basta clicar para prosseguir.

6 - Parabéns sua chave de API ja foi gerada basta clicar em copiar para acessá-la.

---

> **Observação:** 
>
> A chave de API é um elemento crítico para acessar os serviços fornecidos pela Google API. Essa chave funciona como uma *senha* que autentica suas solicitações para usar os modelos Gemini.
> 
> Por esse motivo, deve-se prestar muita atenção para que a mesma não fique disponível publicamente.


## Armazenando uma chave de API

Primeiramente devemos garantir que a chave da API não fique disponível publicamente, é recomendado salvar a sua chave em uma variável de ambiente `GOOGLE_API_KEY`. 

Você pode fazer isso de uma maneira prática criando um arquivo `.env` do seu projeto, para ser carregado em código com o módulo `dotenv`.

Para isso, siga estes passos 

1 - Crie um arquivvo chamado exatamente `".env"`

2 - Abra-o como arquivo de texto e escreva este conteúdo:

```
GOOGLE_API_KEY=<Sua chave da API>.
```

3 - Lembrar de substituir o trecho `<Sua chave da API>` pela sua chave.


> **Atenção:** 
>
> Nunca faça o *commit* do arquivo `.env` para um repositório git público, pois é grande o risco de que algum *hacker* a descubra e use.
> 


## Algumas funções para ajudar

A biblioteca `python-dotenv` serve para lidar com variáveis de ambiente salvas em um arquivo `.env`. Porém, para deixar ainda mais fácil, criamos duas funções:
- **carrega_chave** : Esta função carrega a chave API armazenada no arquivo .env e configura a biblioteca gemini com essa chave.
- **verifica_chave**: Esta função verifica a existência do arquivo .env no sistema.

```python
def carrega_chave(): # Carrega uma chave do arquivo .env
    _ = load_dotenv(find_dotenv())
    chave = os.getenv("GOOGLE_API_KEY")
    gemini.configure(api_key=os.getenv('GOOGLE_API_KEY'))

    print(f"Chave API carregada com sucesso!")
    return chave

def verifica_chave(): # Verifica a existência de um arquivo .env 
    return find_dotenv()
```

Agora, basta fazer a chamada abaixo e... 

```python
chave = carrega_chave()
```

Sua chave estará carregada!


# 2 - Utilizando o serviço (Textual)

## Verificando modelos disponíveis

Primeiramente devemos verificar quais são todos os modelos disponíveis para utilização dsiponibilizados pelo **Google Gemini**.

Para isso basta utilizar o seguinte comando parea receber a lista com os modelos existentes.

```python
modelos = client.get_default_model_client().list_models()
```

Agora a variável **modelos** contém uma lista contendo todos os modelos existentes.

Para listar apenas os modelos generativos, utilize um loop `for` para filtrar os dados em que `model.supported_generation_methods` seja igual a `'generateContent'` como demonstra o código a seguir.

```python
for model in modelos:
    if 'generateContent' in model.supported_generation_methods:
        print(model.name)
```


Principalmente podemos trabalhar com os modelos mais atualizados, que são estes:
 - **gemini-1.5-flash** - O de melhor custo benefício. **Será utilizado nesta publicação**
 - **gemini-1.5-pro** - O melhor modelo (ou seja, que gera o melhor conteúdo).

## Testes

Antes de programar com os modelos, caso queira testar a capacidade de todas as versões possíveis, usando interface gráfica, 
você pode usar o [Google AI Studio](https://aistudio.google.com/)!

## Primeiros passos com o modelo

Este código instancia uma classe para acessar o modelo remotamente (o modelo não é carregado localmente).

```python
model = gemini.GenerativeModel('gemini-1.5-flash')
```

Nesse caso vou solicitar **"faça um resumo em uma linha de:"** a função **statistics.median_high** do python

A fonte desse trecho é da documentação oficial do python: [Documentação do Python: statistics.median_high](https://docs.python.org/pt-br/3/library/statistics.html#statistics.median_high)

Agora, vamos receber nossa primeira mensagem via código:

```python
resposta = model.generate_content("faça um resumo em uma linha de: statistics.median_high(data) Retorna a mediana superior de dados numéricos. Se data for vazio, a exceção StatisticsError é levantada. data pode ser uma sequência ou um iterável. A mediana superior sempre é um membro do conjunto de dados. Quando o número de elementos for ímpar, o valor intermediário é retornado. Se houver um número par de elementos, o maior entre os dois valores centrais é retornado.")
```

Agora, você pode investigar algumas informações da `resposta` recebida com este código:

```python
print(resposta.text) # Resposta textual
print(resposta.usage_metadata) # Informações como número de tokens usados
```

A saída deve ser algo como:
```
`statistics.median_high(data)` retorna o maior dos dois valores médios em um conjunto de dados, ou o único valor médio se o número de elementos for ímpar. 

prompt_token_count: 102
candidates_token_count: 37
total_token_count: 139
```

Logo nesse podemos ja realizar a utilização desse modelo dentro de uma aplicação, exemplo:

Imagine que tenhamos um app que recomenda receitas para o usuário baseando-se nos ingredientes possuídos pelo usuário.
O código abaixo vai ler os ingredientes digitados e montar um **prompt** (ou *query*), ou seja, uma pergunta ou comando para o modelo.


```python
# Pegando o input do suario (simulando uma aplicação).
ingredientes = input("Liste os ingredientes que você possui? ")

# Montando a string completa de 'prompt' ou 'query' (pergunta)
prompt = f"Com os seguintes ingredientes: {ingredientes}, escreva uma receita completa que utilize esses e apenas esses ingredientes."
```

Agora, podemos enviar o *prompt* ao modelo, receber a resposta e imprimir o resultado assim:

```python
# Gera a resposta que estamos buscando
resposta = model.generate_content(prompt)

# Imprime a resposta textual
print(resposta.text) 

# Informações como número de tokens usados
print(resposta.usage_metadata) 
```

Caso o usuário tenha digitado `'ovos, farinha de trigo, leite, açúcar, sal, manteiga'`, a saída seria algo assim:

```
## Bolo Simples de Leite

**Ingredientes:**

* 3 ovos grandes em temperatura ambiente
* 1 xícara (120g) de manteiga sem sal, derretida e levemente resfriada
* 1 xícara (200g) de açúcar granulado
* 2 xícaras (250g) de farinha de trigo
* 1 colher de chá de fermento em pó
* 1/2 colher de chá de bicarbonato de sódio
* 1/2 colher de chá de sal
* 1 xícara (240ml) de leite integral

**Instruções:**

1. Pré-aqueça o forno a 180°C e unte e enfarinhe uma forma redonda com furo no centro de 23cm de diâmetro.
2. Em uma tigela grande, bata os ovos com a manteiga derretida e o açúcar até obter um creme claro e fofo.
3. Em outra tigela, misture a farinha, o fermento, o bicarbonato e o sal.
4. Adicione gradualmente os ingredientes secos à tigela com os ovos, alternando com o leite, começando e terminando com os ingredientes secos. Misture até incorporar todos os ingredientes, mas não bata demais.
5. Despeje a massa na forma preparada e asse por 30 a 35 minutos, ou até que um palito inserido no centro saia limpo.
6. Deixe o bolo esfriar na forma por 10 minutos antes de virá-lo em uma grade para esfriar completamente.

**Dicas:**

* Para um bolo mais úmido, adicione 1/2 xícara de frutas secas ou 1/2 xícara de chocolate picado à massa.
...
prompt_token_count: 33
candidates_token_count: 467
total_token_count: 500
```
