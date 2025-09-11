---
lab:
  title: Detectar objetos em imagens com a Visão Personalizada de IA do Azure
---

# Detectar objetos em imagens com a Visão Personalizada de IA do Azure

Neste exercício, você usa o serviço Visão Personalizada para treinar um modelo de *Detecção de objetos* capaz de detectar e localizar três classes de frutas (maçã, banana e laranja) em uma imagem.

## Criar recursos de Visão Personalizada

Se você já tiver recursos de **Visão Personalizada** para treinamento e previsão em sua assinatura do Azure, poderá usar estes ou uma conta multisserviço existente neste exercício. Ou use as instruções a seguir para criar.

> **Observação**: se você usar uma conta multisserviço, a chave e o ponto de extremidade serão os mesmos para o treinamento e a previsão.

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
   cd mslearn-ai-vision/Labfiles/03-object-detection
    ```

## Criar um projeto de Visão Personalizada

Para treinar um modelo de detecção de objetos, você precisa criar um projeto de Visão Personalizada com base em seu recurso de treinamento. Para fazer isso, você usará o portal de Visão Personalizada.

1. Em uma nova guia do navegador, abra o portal de Visão Personalizada em `https://customvision.ai` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Crie um novo projeto com as seguintes configurações:
    - **Nome**: Detectar Frutas
    - **Descrição**: Detecção de objetos para frutas.
    - **Recurso**: *o recurso de Visão Personalizada criado anteriormente*
    - **Tipos de projeto**: detecção de objetos
    - **Domínios**: Geral
1. Aguarde a criação e abertura do projeto no navegador.

## Adicionar e marcar imagens

Para treinar um modelo de detecção de objeto, você precisa carregar imagens que contenham as classes que serão identificadas pelo modelo e marcá-las para indicar caixas delimitadoras para cada instância de objeto.

1. Baixe as imagens de treinamento de `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/03-object-detection/training-images.zip` e extraia a pasta zip para exibir seu conteúdo. Esta pasta contém imagens de frutas.
1. No portal de Visão Personalizada, no seu projeto de detecção de objetos, selecione **Adicionar imagens** e faça upload de todas as imagens da pasta extraída.
1. Depois de carregar as imagens, selecione a primeira para abri-la.
1. Mantenha o mouse sobre qualquer objeto na imagem até que uma região detectada automaticamente seja exibida, como na imagem abaixo. Em seguida, selecione o objeto e, se necessário, reorganize a região para cercá-lo.

    ![A região padrão de um objeto](../media/object-region.jpg)

    Como alternativa, você pode simplesmente arrastar o objeto para criar uma região.

1. Quando a região envolver o objeto, adicione uma nova marcação com o tipo de objeto apropriado (*maçã*, *banana* ou *laranja*), conforme mostrado aqui:

    ![Um objeto marcado em uma imagem](../media/object-tag.jpg)

1. Selecione e marque os outros objetos na imagem, redimensionando as regiões e adicionando novas marcações conforme o necessário.

    ![Dois objetos marcados em uma imagem](../media/object-tags.jpg)

1. Use o link **>** à direita para ir para a próxima imagem e marcar os objetos. Em seguida, continue trabalhando em toda a coleção de imagens, marcando cada maçã, banana e laranja.

1. Quando terminar de marcar a última imagem, feche o editor de **Detalhes da imagem**. Na página **Imagens de Treinamento**, em **Marcações**, selecione **Marcado** para ver todas as imagens marcadas:

![Imagens marcadas em um projeto](../media/tagged-images.jpg)

## Usar a API de treinamento para carregar imagens

Você pode usar a interface no portal Visão Personalizada para marcar suas imagens, mas muitas equipes de desenvolvimento de IA usam outras ferramentas que geram arquivos contendo informações sobre marcas e regiões de objetos em imagens. Em cenários como este, você pode usar a API de treinamento de Visão Personalizada para carregar imagens marcadas para o projeto.

> **Observação**: neste exercício, você pode optar por usar a API do SDK do **C#** ou **do Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. Clique no ícone *configurações* (&#9881;) no canto superior direito da página **Imagens de treinamento** no portal Visão Personalizada para exibir as configurações do projeto.
1. Em **Geral** (à esquerda), anote a **ID do projeto** que identifica exclusivamente este projeto.
1. À direita, em **Recursos**, observe que a chave e o ponto de extremidade são mostrados. Esses são os detalhes do recurso de *treinamento* (você também pode obter essas informações exibindo o recurso no portal do Azure).
1. De volta ao Portal do Azure, execute o comando `cd C-Sharp/train-detector` ou `cd Python/train-detector` dependendo da sua preferência de idioma.
1. Instale o pacote de Treinamento de Visão Personalizada executando o comando apropriado para sua preferência de idioma:

    **C#**

    ```
   dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
    ```

    **Python**

    ```
   pip install azure-cognitiveservices-vision-customvision==3.1.1
    ```

1. Usando o comando`ls`, você pode exibir o conteúdo da pasta **train-detector**. Nela você encontrará um arquivo para definições de configuração:

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
1. Execute `code tagged-images.json` para abrir o arquivo e examinar o JSON que ele contém. O JSON define uma lista de imagens, cada uma contendo uma ou mais regiões marcadas. Cada região marcada inclui um nome de marca e as coordenadas superior e esquerda e as dimensões de largura e altura da caixa delimitadora que contém o objeto marcado.

    > **Observação**: as coordenadas e dimensões neste arquivo indicam pontos relativos na imagem. Por exemplo, um valor de *altura* de 0,7 indica uma caixa que é 70% da altura da imagem. Algumas ferramentas de marcação geram outros formatos de arquivo nos quais os valores de coordenadas e dimensões representam pixels, polegadas ou outras unidades de medidas.

1. Observe que a pasta **train-detector** contém uma subpasta na qual os arquivos de imagem referenciados no arquivo JSON são armazenados.

1. Observe que a pasta **train-detector** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: train-detector.py

    Abra o arquivo de código e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionTrainingClient** autenticado, que é usado com a ID do projeto para criar uma referência de **Projeto** para o seu projeto.
    - A função **Upload_Images** extrai as informações de região marcada no arquivo JSON e as usa para criar um lote de imagens com regiões, que ele carrega para o projeto.

1. Execute o seguinte comando para executar o programa:
    
    **C#**
    
    ```
   dotnet run
    ```
    
    **Python**
    
    ```
   python train-detector.py
    ```
    
1. Aguarde até que o programa finalize. Em seguida, retorne ao seu navegador e visualize a página **imagens de treinamento** do seu projeto no portal Visão Personalizada (atualize o navegador, se necessário).
1. Verifique se algumas novas imagens marcadas foram adicionadas ao projeto.

## Treinar e testar um modelo

Agora que você marcou as imagens em seu projeto, está tudo pronto para treinar um modelo.

1. No projeto Visão Personalizada, clique em **Treinar** para treinar um modelo de detecção de objeto usando as imagens marcadas. Selecione a opção **Treinamento Rápido**.
1. Aguarde a conclusão do treinamento (pode demorar cerca de dez minutos) e mapeie as métricas de desempenho de *Precisão,*, *Recall* e *mAP* – Essas métricas medem a precisão da previsão do modelo de classificação e devem ser todas altas.
1. No canto superior direito da página, clique em **Teste Rápido** e, na caixa **URL da Imagem**, insira `https://aka.ms/apple-orange` e veja a previsão gerada. Em seguida, feche a janela **Teste Rápido**.

## Publicar o modelo de detecção de objeto

Agora está tudo pronto para publicar seu modelo treinado e poder usá-lo em um aplicativo cliente.

1. No portal Visão Personalizada, na página **Desempenho**, clique em **&#128504; Publicar** para publicar o modelo treinado com as seguintes configurações:
    - **Nome do modelo**: detector de frutas
    - **Recurso de previsão**: *o recurso de **previsão** que você criou anteriormente que termina com "-Prediction" (<u>não</u> o recurso de treinamento)*.
1. No canto superior esquerdo da página **Configurações do Projeto**, clique no ícone *Galeria de Projetos* (&#128065;) para retornar à página inicial do portal Visão Personalizada, onde seu projeto agora está listado.
1. Na página inicial do portal Visão Personalizada, no canto superior direito, clique no ícone *configurações* (&#9881;) para exibir as configurações do serviço Visão Personalizada. Em seguida, em **Recursos**, localize seu recurso de *previsão* que termina com "-Prediction" (<u>não</u> o recurso de treinamento) para determinar seus valores de **Chave** e **ponto de extremidade** (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar o classificador de imagens de um aplicativo cliente

Agora que você publicou o modelo de classificação de imagem, pode usá-lo em um aplicativo cliente. Mais uma vez, você pode optar por usar **C#** ou **Python**.

1. No Portal do Azure, navegue até a pasta **test-detector** executando o comando `cd ../test-detector`.
1. Insira o seguinte comando específico do SDK para instalar o pacote de Previsão da Visão Personalizada:

    **C#**

    ```
   dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
    ```

    **Python**

    ```
   pip install azure-cognitiveservices-vision-customvision==3.1.1
    ```

> **Observação**: o pacote SDK do Python inclui pacotes de treinamento e previsão e pode já estar instalado.

1. Abra o arquivo de configuração do aplicativo cliente (*appsettings.json* para C# ou *.env* para Python) e atualize os valores de configuração que ele contém para refletir o ponto de extremidade e a chave para o recurso de *previsão* de Visão Personalizada, a ID do projeto para o projeto de detecção de objetos e o nome do modelo publicado (que deve ser *fruit-detector*). Salve as alterações e feche o arquivo.
1. Abra o arquivo de código para seu aplicativo cliente (*Program.cs* para C#, *test-detector.py* para Python) e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionPredictionClient** autenticado.
    - O objeto de cliente de previsão é usado para obter previsões de detecção de objetos para a imagem **produce.jpg**, especificando a ID do projeto e o nome do modelo na solicitação. As regiões marcadas previstas são então desenhadas na imagem e o resultado é salvo como **output.jpg**.
1. Feche o editor de código e insira o seguinte comando para executar o programa:

    **C#**

    ```
   dotnet run
    ```

    **Python**

    ```
   python test-detector.py
    ```

1. Depois que o programa for concluído, na barra de ferramentas do Cloud Shell, selecione **Carregar/Baixar arquivos** e, em seguida, **Baixar**. Na nova caixa de diálogo, insira o seguinte caminho de arquivo e selecione **Baixar**:

    **C#**
   
    ```
   mslearn-ai-vision/Labfiles/03-object-detection/C-Sharp/test-detector/output.jpg
    ```

    **Python**
   
    ```
   mslearn-ai-vision/Labfiles/03-object-detection/Python/test-detector/output.jpg
    ```

1. Exiba o arquivo **output.jpg** resultante para ver os objetos detectados na imagem.

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais.

1. Abra o portal do Azure em `https://portal.azure.com`,e na barra de pesquisa superior, procure os recursos criados neste laboratório.

1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso. Como alternativa, você pode excluir todo o grupo de recursos para limpar todos os recursos ao mesmo tempo.
   
## Mais informações

Para obter mais informações sobre a detecção de objetos com o serviço Visão Personalizada, consulte a [documentação da Visão Personalizada](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
