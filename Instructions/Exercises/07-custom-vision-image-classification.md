---
lab:
  title: Classificar imagens com a Visão Personalizada de IA do Azure
---

# Classificar imagens com a Visão Personalizada de IA do Azure

O serviço de **Visão Personalizada de IA do Azure** permite que você crie modelos de Pesquisa Visual Computacional treinados com suas próprias imagens. Você pode usar para treinar os modelos de *classificação de imagem* e *detecção de objetos*, que podem ser publicados e consumidos em aplicativos.

Neste exercício, você usa o serviço Visão Personalizada para treinar um modelo de classificação de imagem capaz de identificar três classes de frutas (maçã, banana e laranja).

## Clone o repositório para este curso

Se você ainda não clonou o repositório de código **mslearn-ai-vision** para o ambiente em que está trabalhando neste laboratório, siga estas etapas para fazer isso. Caso contrário, abra a pasta clonada no Visual Studio Code.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**. Se você for surpreendido com a mensagem *Detectado um projeto de função do Azure na pasta*, você pode fechar essa mensagem com segurança.

## Criar recursos de Visão Personalizada

Antes de treinar um modelo, você precisará de recursos do Azure para *treinamento* e *previsão*. Você pode criar recursos de **Visão Personalizada** para cada uma dessas tarefas ou pode criar um único recurso de **Serviços de IA do Azure** e usá-lo para qualquer uma (ou ambas).

Neste exercício, você criará recursos de **Visão Personalizada** para treinamento e previsão para que possa gerenciar o acesso e os custos dessas cargas de trabalho separadamente.

1. Em uma nova guia do navegador, abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada com sua assinatura do Azure.
2. Selecione o botão **&#65291;Criar um recurso**, procure *Visão Personalizada* e crie um recurso de **Visão Personalizada** com as seguintes configurações:
    - **Criar opções**: ambos
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se você estiver usando uma assinatura restrita, talvez não tenha permissão para criar um novo grupo de recursos; use o que foi fornecido)*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço de treinamento**: F0
    - **Tipo de preço da previsão**: F0

    > **Observação**: se você já tiver um serviço de visão personalizada F0 em sua assinatura, selecione **S0** para este.

3. Aguarde até que os recursos sejam criados e, em seguida, exiba os detalhes da implantação e observe que dois recursos de Visão Personalizada são provisionados; um para treinamento e outro para previsão (evidente pelo sufixo **-Prediction**). Você pode exibir navegando até o grupo de recursos onde os criou.

> **Importante**: cada recurso tem seu próprio *ponto de extremidade* e *chaves*, que são usados para gerenciar o acesso do seu código. Para treinar um modelo de classificação de imagem, seu código deve usar o recurso de *treinamento* (com seu ponto de extremidade e chave) e, para usar o modelo treinado para prever classes de imagem, seu código deve usar o recurso de *previsão* (com seu ponto de extremidade e chave).

## Criar um projeto de Visão Personalizada

Para treinar um modelo de classificação de imagens, você precisa criar um projeto de Visão Personalizada com base em seu recurso de treinamento. Para fazer isso, você usará o portal de Visão Personalizada.

1. No Visual Studio Code, veja as imagens do treinamento na pasta **Labfiles/07-custom-vision-image-classification/training-images** em que você clonou o repositório. Esta pasta contém subpastas de imagens de maçã, banana e laranja.
2. Em outra guia do navegador, abra o portal de Visão Personalizada em `https://customvision.ai`. Caso solicitado, entre usando a conta Microsoft associada à sua assinatura do Azure e concorde com os termos de serviço.
3. No portal de Visão Personalizada, crie um projeto com as seguintes configurações:
    - **Nome**: classificar frutas
    - **Descrição**: classificação de imagem de frutas
    - **Recurso**: *o recurso de Visão Personalizada criado anteriormente*
    - **Tipos de Projeto**: Classificação
    - **Tipos de classificação**: Multiclasse (tag única por imagem)
    - **Domínios**: comida
4. No novo projeto, clique em **\[+\]Adicionar imagens** e selecione todos os arquivos na pasta **LabFiles/07-custom-vision-image-classification/training-images/apple** visualizada anteriormente. Em seguida, carregue os arquivos de imagem, especificando a tag *maçã*, desta forma:

![Carregar maçã com tag maçã](../media/upload_apples.jpg)
   
5. Repita a etapa anterior para carregar as imagens na pasta **banana** com a tag *banana* e as imagens na pasta **laranja** com a tag *laranja*.
6. Explore as imagens que você carregou no projeto de Visão Personalizada – deve haver 15 imagens de cada classe, desta forma:

![Imagens de frutas com tags – 15 maçãs, 15 bananas e 15 laranjas](../media/fruit.jpg)
    
7. No projeto de Visão Personalizada, acima das imagens, clique em **Treinar** para treinar um modelo de classificação usando as imagens com tag. Selecione a opção de **Treinamento Rápido** e aguarde a conclusão da iteração de treinamento (isso pode levar mais ou menos um minuto).
8. Quando a iteração do modelo tiver sido treinada, revise as métricas de desempenho de *Precisão*, *Recall* e *PA* – elas medem a precisão de predição do modelo de classificação e devem ser todas altas.

> **Observação**: as métricas de desempenho são baseadas em um limite de probabilidade de 50% para cada previsão (em outras palavras, se o modelo calcular uma probabilidade de 50% ou mais de que uma imagem seja de uma classe específica, essa classe será prevista). Você pode ajustar isso no canto superior esquerdo da página.

## Testar o modelo

Agora que você treinou o modelo, pode testá-lo.

1. Acima das métricas de desempenho, clique em **Início Rápido**.
2. Na caixa **URL da imagem**, digite `https://aka.ms/apple-image` e clique em &#10132;
3. Exiba as previsões retornadas por seu modelo – a pontuação de probabilidade para *maçã* deve ser a mais alta, assim:

![Uma imagem com uma previsão de classe de maçã](../media/test-apple.jpg)

4. Feche a janela **Teste Rápido**.

## Exibir as configurações do projeto

O projeto que você criou recebeu um identificador exclusivo, que você precisará especificar em qualquer código que interaja com ele.

1. Clique no ícone *configurações* (&#9881;) no canto superior direito da página **Desempenho** para exibir as configurações do projeto.
2. Em **Geral** (à esquerda), anote a **ID do projeto** que identifica exclusivamente este projeto.
3. À direita, em **Recursos**, observe que a chave e o ponto de extremidade são mostrados. Esses são os detalhes do recurso de *treinamento* (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar a API de *treinamento*

O portal de Visão Personalizada fornece uma interface de usuário conveniente que você pode usar para carregar e marcar imagens e treinar modelos. No entanto, em alguns cenários, convém automatizar o treinamento de modelo usando a API de treinamento de Visão Personalizada.

> **Observação**: neste exercício, você pode optar por usar a API do SDK do **C#** ou **do Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. No Visual Studio Code, no painel **Explorar**, navegue até a pasta **07-custom-vision-image-classification** e expanda a pasta **C-Sharp** ou **Python** dependendo da sua preferência de linguagem.
2. Clique com o botão direito do mouse na pasta **train-classifier** e abra um terminal integrado. Em seguida, instale o pacote de treinamento de visão personalizada executando o comando apropriado para sua preferência de linguagem:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

3. Exiba o conteúdo da pasta **train-classifier** e observe que ela contém um arquivo para definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

    Abra o arquivo de configuração e atualize os valores de configuração que ele contém para refletir o ponto de extremidade e a chave para seu recurso de *treinamento* de Visão Personalizada e a ID do projeto de classificação criado anteriormente. Salve suas alterações.
4. Observe que a pasta **train-classifier** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: train-classifier.py

    Abra o arquivo de código e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionTrainingClient** autenticado, que é usado com a ID do projeto para criar uma referência de **Projeto** para o seu projeto.
    - A função **Upload_Images** recupera as marcas definidas no projeto de visão personalizada e, em seguida, carrega arquivos de imagem de pastas nomeadas correspondentes para o projeto, atribuindo a ID de marca apropriada.
    - A função **Train_Model** cria uma nova iteração de treinamento para o projeto e aguarda a conclusão do treinamento.
5. Retorne o terminal integrado para a pasta **train-classifier** e digite o seguinte comando para executar o programa:

**C#**

```
dotnet run
```

**Python**

```
python train-classifier.py
```
    
6. Aguarde até que o programa finalize. Em seguida, retorne ao seu navegador e visualize a página **imagens de treinamento** do seu projeto no portal Visão Personalizada (atualize o navegador, se necessário).
7. Verifique se algumas novas imagens marcadas foram adicionadas ao projeto. Em seguida, exiba a página **Desempenho** e verifique se uma nova iteração foi criada.

## Publicar o modelo de classificação de imagem

Agora está tudo pronto para publicar seu modelo treinado e poder usá-lo em um aplicativo cliente.

1. No portal Visão Personalizada, na página **Desempenho**, clique em **&#128504; Publicar** para publicar o modelo treinado com as seguintes configurações:
    - **Nome do modelo**: fruit-classifier
    - **Recurso de previsão**: *o recurso de **previsão** que você criou anteriormente que termina com "-Prediction" (<u>não</u> o recurso de treinamento)*.
2. No canto superior esquerdo da página **Configurações do Projeto**, clique no ícone *Galeria de Projetos* (&#128065;) para retornar à página inicial do portal Visão Personalizada, onde seu projeto agora está listado.
3. Na página inicial do portal Visão Personalizada, no canto superior direito, clique no ícone *configurações* (&#9881;) para exibir as configurações do serviço Visão Personalizada. Em seguida, em **Recursos**, localize seu recurso de *previsão* que termina com "-Prediction" (<u>não</u> o recurso de treinamento) para determinar seus valores de **Chave** e **ponto de extremidade** (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar o classificador de imagens de um aplicativo cliente

Agora que você publicou o modelo de classificação de imagem, pode usá-lo em um aplicativo cliente. Mais uma vez, você pode optar por usar **C#** ou **Python**.

1. No Visual Studio Code, na pasta **07-custom-vision-image-classification**, na subpasta da linguagem de sua preferência (**C-Sharp** ou **Python**), à direita da pasta **test-classifier** e abra um terminal integrado. Em seguida, insira o seguinte comando específico do SDK para instalar o pacote de previsão de visão personalizada:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

> **Observação**: o pacote SDK do Python inclui pacotes de treinamento e previsão e pode já estar instalado.

2. Expanda a pasta **test-classifier** para exibir os arquivos que ela contém, que são usados para implementar um aplicativo cliente de teste para seu modelo de classificação de imagem.
3. Abra o arquivo de configuração do aplicativo cliente (*appsettings.json* para C# ou *.env* para Python) e atualize os valores de configuração que ele contém para refletir o ponto de extremidade e a chave do recurso de previsão da *Visão Personalizada*, a ID do projeto de classificação e o nome do modelo publicado (que deve ser *fruit-classifier*). Salve suas alterações.
4. Abra o arquivo de código para seu aplicativo cliente (*Program.cs* para C#, *test-classification.py* para Python) e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionPredictionClient** autenticado.
    - O objeto de cliente de previsão é usado para prever uma classe para cada imagem na pasta **test-images**, especificando a ID do projeto e o nome do modelo para cada solicitação. Cada previsão inclui uma probabilidade para cada classe possível, e apenas as tags previstas com uma probabilidade maior que 50% são exibidas.
5. Retorne o terminal integrado para a pasta **test-classifier** e digite o seguinte comando específico do SDK para executar o programa:

**C#**

```
dotnet run
```

**Python**

```
python test-classifier.py
```

6. Exiba o rótulo (marcação) e as pontuações de probabilidade para cada previsão. Você pode exibir as imagens na pasta **test-images** para verificar se o modelo as classificou corretamente.

## Mais informações

Para obter mais informações sobre a classificação de imagem com o serviço Visão Personalizada, consulte a [documentação da Visão Personalizada](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
