---
lab:
  title: Classificar imagens com a Visão Personalizada de IA do Azure
---

# Classificar imagens com a Visão Personalizada de IA do Azure

O serviço de **Visão Personalizada de IA do Azure** permite que você crie modelos de Pesquisa Visual Computacional treinados com suas próprias imagens. Você pode usar para treinar os modelos de *classificação de imagem* e *detecção de objetos*, que podem ser publicados e consumidos em aplicativos.

Neste exercício, você usa o serviço Visão Personalizada para treinar um modelo de classificação de imagem capaz de identificar três classes de frutas (maçã, banana e laranja).

## Criar recursos de Visão Personalizada

Antes de treinar um modelo, você precisará de recursos do Azure para *treinamento* e *previsão*. Você pode criar recursos de **Visão Personalizada** para cada uma dessas tarefas ou pode criar um único recurso de **Serviços de IA do Azure** e usá-lo para qualquer uma (ou ambas).

Neste exercício, você criará recursos de **Visão Personalizada** para treinamento e previsão para que possa gerenciar o acesso e os custos dessas cargas de trabalho separadamente.

1. Em uma nova guia do navegador, abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada com sua assinatura do Azure.
1. Selecione o botão **&#65291;Criar um recurso**, procure *Visão Personalizada* e crie um recurso de **Visão Personalizada** com as seguintes configurações:
    - **Criar opções**: ambos
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se você estiver usando uma assinatura restrita, talvez não tenha permissão para criar um novo grupo de recursos; use o que foi fornecido)*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço de treinamento**: F0
    - **Tipo de preço da previsão**: F0

    > **Observação**: se você já tiver um serviço de visão personalizada F0 em sua assinatura, selecione **S0** para este.

1. Aguarde até que os recursos sejam criados e, em seguida, exiba os detalhes da implantação e observe que dois recursos de Visão Personalizada são provisionados; um para treinamento e outro para previsão (evidente pelo sufixo **-Prediction**). Você pode exibir navegando até o grupo de recursos onde os criou.

> **Importante**: cada recurso tem seu próprio *ponto de extremidade* e *chaves*, que são usados para gerenciar o acesso do seu código. Para treinar um modelo de classificação de imagem, seu código deve usar o recurso de *treinamento* (com seu ponto de extremidade e chave) e, para usar o modelo treinado para prever classes de imagem, seu código deve usar o recurso de *previsão* (com seu ponto de extremidade e chave).

## Clone o repositório para este curso

Você desenvolverá seu código usando o Cloud Shell no Portal do Azure. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: Se você já clonou o repositório **mslearn-ai-vision** recentemente, pode ignorar essa tarefa. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. No Portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-vision/Labfiles/07-custom-vision-image-classification
    ```
    
## Criar um projeto de Visão Personalizada

Para treinar um modelo de classificação de imagens, você precisa criar um projeto de Visão Personalizada com base em seu recurso de treinamento. Para fazer isso, você usará o portal de Visão Personalizada.

1. Baixe as imagens de treinamento de `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/07-custom-vision-image-classification/training-images.zip` e extraia a pasta zip para exibir seu conteúdo. Esta pasta contém subpastas de imagens de maçã, banana e laranja.
1. Em outra guia do navegador, abra o portal de Visão Personalizada em `https://customvision.ai`. Caso solicitado, entre usando a conta Microsoft associada à sua assinatura do Azure e concorde com os termos de serviço.
1. No portal de Visão Personalizada, crie um projeto com as seguintes configurações:
    - **Nome**: classificar frutas
    - **Descrição**: classificação de imagem de frutas
    - **Recurso**: *o recurso de Visão Personalizada criado anteriormente*
    - **Tipos de Projeto**: Classificação
    - **Tipos de classificação**: Multiclasse (tag única por imagem)
    - **Domínios**: comida
1. No novo projeto, clique em **\[+\] Adicionar imagens** e selecione todos os arquivos na pasta **training-images/apple** que você extraiu anteriormente. Em seguida, carregue os arquivos de imagem, especificando a tag *maçã*, desta forma:

![Carregar maçã com tag maçã](../media/upload_apples.jpg)
   
1. Repita a etapa anterior para carregar as imagens na pasta **banana** com a tag *banana* e as imagens na pasta **laranja** com a tag *laranja*.
1. Explore as imagens que você carregou no projeto de Visão Personalizada – deve haver 15 imagens de cada classe, desta forma:

![Imagens de frutas com tags – 15 maçãs, 15 bananas e 15 laranjas](../media/fruit.jpg)
    
1. No projeto de Visão Personalizada, acima das imagens, clique em **Treinar** para treinar um modelo de classificação usando as imagens com tag. Selecione a opção de **Treinamento Rápido** e aguarde a conclusão da iteração de treinamento (isso pode levar mais ou menos um minuto).
1. Quando a iteração do modelo tiver sido treinada, revise as métricas de desempenho de *Precisão*, *Recall* e *PA* – elas medem a precisão de predição do modelo de classificação e devem ser todas altas.

> **Observação**: as métricas de desempenho são baseadas em um limite de probabilidade de 50% para cada previsão (em outras palavras, se o modelo calcular uma probabilidade de 50% ou mais de que uma imagem seja de uma classe específica, essa classe será prevista). Você pode ajustar isso no canto superior esquerdo da página.

## Testar o modelo

Agora que você treinou o modelo, pode testá-lo.

1. Acima das métricas de desempenho, clique em **Início Rápido**.
1. Na caixa **URL da imagem**, digite `https://aka.ms/apple-image` e clique em &#10132;
1. Exiba as previsões retornadas por seu modelo – a pontuação de probabilidade para *maçã* deve ser a mais alta, assim:

![Uma imagem com uma previsão de classe de maçã](../media/test-apple.jpg)

1. Feche a janela **Teste Rápido**.

## Exibir as configurações do projeto

O projeto que você criou recebeu um identificador exclusivo, que você precisará especificar em qualquer código que interaja com ele.

1. Clique no ícone *configurações* (&#9881;) no canto superior direito da página **Desempenho** para exibir as configurações do projeto.
1. Em **Geral** (à esquerda), anote a **ID do projeto** que identifica exclusivamente este projeto.
1. À direita, em **Recursos**, observe que a chave e o ponto de extremidade são mostrados. Esses são os detalhes do recurso de *treinamento* (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar a API de *treinamento*

O portal de Visão Personalizada fornece uma interface de usuário conveniente que você pode usar para carregar e marcar imagens e treinar modelos. No entanto, em alguns cenários, convém automatizar o treinamento de modelo usando a API de treinamento de Visão Personalizada.

> **Observação**: neste exercício, você pode optar por usar a API do SDK do **C#** ou **do Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. De volta ao Portal do Azure, execute o comando `cd C-Sharp/train-classifier` ou `cd Python/train-classifier` dependendo da sua preferência de idioma.
1. Instale o pacote de Treinamento de Visão Personalizada executando o comando apropriado para sua preferência de idioma:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-customvision==3.1.0
    ```

1. Usando o comando `ls`, você pode exibir o conteúdo da pasta **train-classifier** e observar que ela contém um arquivo para configurações:
    - **C#**: appsettings.json
    - **Python**: .env

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, atualize os valores de configuração que ele contém para refletir o **ponto de extremidade** e a **chave** de autenticação para seu recurso de *treinamento* de Visão Personalizada e a ID do projeto para o projeto de detecção de objetos criado anteriormente.
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.
1. Observe que a pasta **train-classifier** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: train-classifier.py

    Abra o arquivo de código e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionTrainingClient** autenticado, que é usado com a ID do projeto para criar uma referência de **Projeto** para o seu projeto.
    - A função **Upload_Images** recupera as marcas definidas no projeto de visão personalizada e, em seguida, carrega arquivos de imagem de pastas nomeadas correspondentes para o projeto, atribuindo a ID de marca apropriada.
    - A função **Train_Model** cria uma nova iteração de treinamento para o projeto e aguarda a conclusão do treinamento.
1. Feche o editor de código e insira o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python train-classifier.py
    ```
    
1. Aguarde até que o programa finalize. Em seguida, retorne ao seu navegador e visualize a página **imagens de treinamento** do seu projeto no portal Visão Personalizada (atualize o navegador, se necessário).
1. Verifique se algumas novas imagens marcadas foram adicionadas ao projeto. Em seguida, exiba a página **Desempenho** e verifique se uma nova iteração foi criada.

## Publicar o modelo de classificação de imagem

Agora está tudo pronto para publicar seu modelo treinado e poder usá-lo em um aplicativo cliente.

1. No portal Visão Personalizada, na página **Desempenho**, clique em **&#128504; Publicar** para publicar o modelo treinado com as seguintes configurações:
    - **Nome do modelo**: fruit-classifier
    - **Recurso de previsão**: *o recurso de **previsão** que você criou anteriormente que termina com "-Prediction" (<u>não</u> o recurso de treinamento)*.
1. No canto superior esquerdo da página **Configurações do Projeto**, clique no ícone *Galeria de Projetos* (&#128065;) para retornar à página inicial do portal Visão Personalizada, onde seu projeto agora está listado.
1. Na página inicial do portal Visão Personalizada, no canto superior direito, clique no ícone *configurações* (&#9881;) para exibir as configurações do serviço Visão Personalizada. Em seguida, em **Recursos**, localize seu recurso de *previsão* que termina com "-Prediction" (<u>não</u> o recurso de treinamento) para determinar seus valores de **Chave** e **ponto de extremidade** (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar o classificador de imagens de um aplicativo cliente

Agora que você publicou o modelo de classificação de imagem, pode usá-lo em um aplicativo cliente. Mais uma vez, você pode optar por usar **C#** ou **Python**.

1. De volta ao Cloud Shell, execute o comando `cd ../test-classifier` e insira o seguinte comando específico do SDK para instalar o pacote de Previsão da Visão Personalizada:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-customvision==3.1.0
    ```

> **Observação**: o pacote SDK do Python inclui pacotes de treinamento e previsão e pode já estar instalado.

1. Usando o comando `ls`, você pode exibir o conteúdo da pasta **test-classifier**, que é usada para implementar um aplicativo cliente de teste para o modelo de classificação de imagem.
1. Abra o arquivo de configuração do aplicativo cliente (*appsettings.json* para C# ou *.env* para Python) e atualize os valores de configuração que ele contém para refletir o ponto de extremidade e a chave do recurso de previsão da *Visão Personalizada*, a ID do projeto de classificação e o nome do modelo publicado (que deve ser *fruit-classifier*). Salve as alterações e feche o editor de código.
1. Abra o arquivo de código para seu aplicativo cliente (*Program.cs* para C#, *test-classification.py* para Python) e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionPredictionClient** autenticado.
    - O objeto de cliente de previsão é usado para prever uma classe para cada imagem na pasta **test-images**, especificando a ID do projeto e o nome do modelo para cada solicitação. Cada previsão inclui uma probabilidade para cada classe possível, e apenas as tags previstas com uma probabilidade maior que 50% são exibidas.
1. Feche o editor de código e insira o seguinte comando específico do SDK para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python test-classifier.py
    ```

1. Exiba o rótulo (marcação) e as pontuações de probabilidade para cada previsão. Você pode baixar as imagens na pasta **test-images** para verificar se o modelo as classificou corretamente.

1. Na barra de ferramentas do Cloud Shell, selecione **Carregar/Baixar arquivos** e, em seguida, **Baixar**. Na nova caixa de diálogo, insira o seguinte caminho de arquivo e selecione **Baixar**:

    **C#**
   
    ```
   mslearn-ai-vision/Labfiles/07-custom-vision-image-classification/C-Sharp/test-classifier/test-images/IMG_TEST_1.jpg
    ```

    **Python**
   
    ```
   mslearn-ai-vision/Labfiles/07-custom-vision-image-classification/Python/test-classifier/test-images/IMG_TEST_1.jpg
    ```

   Você também pode baixar `IMG_TEST_2.jpg` e `IMG_TEST_3.jpg` usando o mesmo caminho.

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais.

1. Abra o portal do Azure em `https://portal.azure.com`,e na barra de pesquisa superior, procure os recursos criados neste laboratório.

1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso. Como alternativa, você pode excluir todo o grupo de recursos para limpar todos os recursos ao mesmo tempo.

## Mais informações

Para obter mais informações sobre a classificação de imagem com o serviço Visão Personalizada, consulte a [documentação da Visão Personalizada](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
