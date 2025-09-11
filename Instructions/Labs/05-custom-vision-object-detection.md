---
lab:
  title: Detectar objetos em imagens
  description: Use o serviço Visão Personalizada de IA do Azure para treinar um modelo de detecção de objetos.
---

# Detectar objetos em imagens

O serviço de **Visão Personalizada de IA do Azure** permite que você crie modelos de Pesquisa Visual Computacional treinados com suas próprias imagens. Você pode usar para treinar os modelos de *classificação de imagem* e *detecção de objetos*, que podem ser publicados e consumidos em aplicativos.

Neste exercício, você usa o serviço Visão Personalizada para treinar um modelo de *Detecção de objetos* capaz de detectar e localizar três classes de frutas (maçã, banana e laranja) em uma imagem.

Embora este exercício seja baseado no SDK de Visão Personalizada do Azure para Python, você pode desenvolver aplicativos de visão usando vários SDKs específicos de linguagem; incluindo:

* [Visão Personalizada do Azure para JavaScript (treinamento)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training)
* [Visão Personalizada do Azure para JavaScript (previsão)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction)
* [Visão Personalizada do Azure para Microsoft .NET (treinamento)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
* [Visão Personalizada do Azure para Microsoft .NET (previsão)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)
* [Visão Personalizada do Azure para Java (treinamento)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-training/1.1.0-preview.2/jar)
* [Visão Personalizada do Azure para Java (previsão)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-prediction/1.1.0-preview.2/jar)

Este exercício levará, aproximadamente, **45** minutos.

## Criar recursos de Visão Personalizada

Antes de treinar um modelo, você precisará de recursos do Azure para *treinamento* e *previsão*. Você pode criar recursos de **Visão Personalizada** para cada uma dessas tarefas ou criar um único recurso e usá-lo para ambas. Neste exercício, você criará recursos de **Visão Personalizada** para treinamento e previsão.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre usando suas credenciais do Azure. Feche todas as mensagens de boas-vindas ou dicas exibidas.
1. Selecione **Criar um recurso**.
1. Na barra de pesquisa, pesquise `Custom Vision`, selecione **Visão Personalizada** e crie o recurso com as seguintes configurações:
    - **Criar opções**: ambos
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *Um nome válido para seu recurso de Visão Personalizada*
    - **Tipo de preço de treinamento**: F0
    - **Tipo de preço da previsão**: F0

1. Crie o recurso, aguarde a conclusão da implantação e exiba os detalhes da implantação. Observe que dois recursos de Visão Personalizada são provisionados: um para treinamento e outro para previsão.

    > **Observação**: Cada recurso tem seu próprio *ponto de extremidade* e *chaves*, que são usadas para gerenciar o acesso do seu código. Para treinar um modelo de classificação de imagem, seu código deve usar o recurso de *treinamento* (com seu ponto de extremidade e chave) e, para usar o modelo treinado para prever classes de imagem, seu código deve usar o recurso de *previsão* (com seu ponto de extremidade e chave).

1. Quando os recursos tiverem sido implantados, vá para o grupo de recursos para exibi-los. Você deverá ver dois recursos de Visão Personalizada, sendo um com o sufixo ***-Previsão***.

## Criar um projeto de Visão Personalizada no portal de Visão Personalizada

Para treinar um modelo de detecção de objetos, você precisa criar um projeto de Visão Personalizada com base em seu recurso de treinamento. Para fazer isso, você usará o portal de Visão Personalizada.

1. Abra uma nova guia do navegador (mantendo a guia do portal do Azure aberta – você retornará a ela mais tarde).
1. Em uma nova guia do navegador, abra o [portal de Visão Personalizada](https://customvision.ai) em `https://customvision.ai`. Se solicitado, entre usando suas credenciais do Azure e concorde com os termos de serviço.
1. Crie um novo projeto com as seguintes configurações:
    - **Nome**: `Detect Fruit`
    - **Descrição**: `Object detection for fruit.`
    - **Recurso**: *Seu recurso de Visão Personalizada*
    - **Tipos de projeto**: detecção de objetos
    - **Domínios**: Geral
1. Aguarde a criação e abertura do projeto no navegador.

## Carregar e marcar imagens

Agora que você tem um projeto de detecção de objetos, pode carregar e marcar imagens para treinar um modelo.

### Carregar e marcar imagens no portal de Visão Personalizada

O portal de Visão Personalizada inclui ferramentas visuais que você pode usar para carregar imagens e marcar regiões dentro delas que contêm vários tipos de objeto.

1. Em uma nova guia do navegador, baixe as [imagens de treinamento](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/object-detection/training-images.zip) de `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/object-detection/training-images.zip` e extraia a pasta zip para exibir seu conteúdo. Esta pasta contém imagens de frutas.
1. No portal de Visão Personalizada, no seu projeto de detecção de objetos, selecione **Adicionar imagens** e faça upload de todas as imagens da pasta extraída.
1. Depois de carregar as imagens, selecione a primeira para abri-la.
1. Mantenha o mouse sobre qualquer objeto na imagem até que uma região detectada automaticamente seja exibida, como na imagem abaixo. Em seguida, selecione o objeto e, se necessário, reorganize a região para cercá-lo.

    ![Captura de tela da região padrão de um objeto.](../media/object-region.jpg)

    Como alternativa, você pode simplesmente arrastar o objeto para criar uma região.

1. Quando a região envolver o objeto, adicione uma nova marcação com o tipo de objeto apropriado (*maçã*, *banana* ou *laranja*), conforme mostrado aqui:

    ![Captura de tela de um objeto marcado em uma imagem.](../media/object-tag.jpg)

1. Selecione e marque os outros objetos na imagem, redimensionando as regiões e adicionando novas marcações conforme o necessário.

    ![Captura de tela de dois objetos marcados em uma imagem.](../media/object-tags.jpg)

1. Use o link **>** à direita para ir para a próxima imagem e marcar os objetos. Em seguida, continue trabalhando em toda a coleção de imagens, marcando cada maçã, banana e laranja.

1. Quando terminar de marcar a última imagem, feche o editor de **Detalhes da imagem**. Na página **Imagens de Treinamento**, em **Marcações**, selecione **Marcado** para ver todas as imagens marcadas:

![Captura de tela de imagens marcadas em um projeto.](../media/tagged-images.jpg)

### Usar o SDK da Visão Personalizada para carregar imagens

Você pode usar a interface no portal Visão Personalizada para marcar suas imagens, mas muitas equipes de desenvolvimento de IA usam outras ferramentas que geram arquivos contendo informações sobre marcas e regiões de objetos em imagens. Em cenários como este, você pode usar a API de treinamento de Visão Personalizada para carregar imagens marcadas para o projeto.

1. Clique no ícone *configurações* (&#9881;) no canto superior direito da página **Imagens de treinamento** no portal Visão Personalizada para exibir as configurações do projeto.
1. Em **Geral** (à esquerda), anote a **ID do projeto** que identifica exclusivamente este projeto.
1. À direita, em **Recursos**, observe que a **Chave** e o **Ponto de extremidade** são mostrados. Esses são os detalhes do recurso de *treinamento* (você também pode obter essas informações exibindo o recurso no portal do Azure).
1. Retorne à guia do navegador que contém o portal do Azure (mantendo a guia do portal de Visão Personalizada aberta – você retornará a ela mais tarde).
1. No portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

    > **Observação**: Se o portal solicitar que você selecione um armazenamento para persistir seus arquivos, escolha **Nenhuma conta de armazenamento necessária**, selecione a assinatura que você está usando e pressione **Aplicar**.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. Redimensione o painel do Cloud Shell para conseguir ver mais dele.

    > **Dica**" Você pode redimensionar o painel arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Depois que o repositório tiver sido clonado, use o seguinte comando para navegar até os arquivos de código do aplicativo:

    ```
   cd mslearn-ai-vision/Labfiles/object-detection/python/train-detector
   ls -a -l
    ```

    A pasta contém a configuração do aplicativo e os arquivos de código para seu aplicativo. Ele também contém um arquivo **tagged-images.json** que contém coordenadas de caixa delimitadora para objetos em várias imagens e uma subpasta **/images**, que contém as imagens.

1. Instale o pacote do SDK da Visão Personalizada da IA do Azure para treinamento e quaisquer outros pacotes necessários executando os seguintes comandos:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. Insira o seguinte comando para editar o arquivo de configuração do aplicativo:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, atualize os valores de configuração para refletir o **Ponto de extremidade** e a **Chave** de autenticação do seu recurso de *treinamento* da Visão Personalizada, e a **ID do Projeto** do projeto de Visão Personalizada que você criou anteriormente.
1. Depois de substituir os espaços reservados, no editor de código, use o comando **CTRL+S** para salvar as alterações e então use **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.
1. Na linha de comando do Cloud Shell, digite o seguinte comando para abrir o arquivo **tagged-images.json** e ver as informações de marcação dos arquivos de imagem na subpasta **/images**:

    ```
   code tagged-images.json
    ```
    
     O JSON define uma lista de imagens, cada uma contendo uma ou mais regiões marcadas. Cada região marcada inclui um nome de marca e as coordenadas superior e esquerda e as dimensões de largura e altura da caixa delimitadora que contém o objeto marcado.

    > **Observação**: as coordenadas e dimensões neste arquivo indicam pontos relativos na imagem. Por exemplo, um valor de *altura* de 0,7 indica uma caixa que é 70% da altura da imagem. Algumas ferramentas de marcação geram outros formatos de arquivo nos quais os valores de coordenadas e dimensões representam pixels, polegadas ou outras unidades de medidas.

1. Feche o arquivo JSON sem salvar nenhuma alteração (*CTRL_Q*).

1. Na linha de comando do Cloud Shell, insira o seguinte comando para abrir o arquivo de código do aplicativo cliente:

    ```
   code add-tagged-images.py
    ```

1. Observe os seguintes detalhes no arquivo de código:
    - Os namespaces do SDK de Visão Personalizada da IA do Azure são importados.
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionTrainingClient** autenticado, que é usado com a ID do projeto para criar uma referência de **Projeto** para o seu projeto.
    - A função **Upload_Images** extrai as informações de região marcada no arquivo JSON e as usa para criar um lote de imagens com regiões, que ele carrega para o projeto.

1. Feche o editor de código (*CTRL+Q*) e insira o seguinte comando para executar o programa:

    ```
   python add-tagged-images.py
    ```

1. Aguarde até que o programa finalize.
1. Volte para a guia do navegador que contém o portal de Visão Personalizada (mantendo a guia do Cloud Shell do Azure aberta) e visualize a página **Treinamento de Imagens** do seu projeto (atualizando o navegador, se necessário).
1. Verifique se algumas novas imagens marcadas foram adicionadas ao projeto.

## Treinar e testar um modelo

Agora que você marcou as imagens em seu projeto, está tudo pronto para treinar um modelo.

1. No projeto Visão Personalizada, clique em **Treinar** (&#9881;<sub>&#9881;</sub>) para treinar um modelo de detecção de objetos usando as imagens marcadas. Selecione a opção **Treinamento Rápido**.
1. Aguarde até que o treinamento seja concluído (pode levar dez minutos ou mais).

    > **Dica**: O Azure Cloud Shell tem um tempo limite de inatividade de 20 minutos, após o qual a sessão é abandonada. Enquanto espera o treinamento ser concluído, retorne ocasionalmente ao Cloud Shell e digite um comando qualquer, como `ls`, para manter a sessão ativa.

1. No portal de Visão Personalizada, quando o treinamento for concluído, revise as métricas de desempenho *Precision*, *Recall* e *mAP* — elas medem a precisão das previsões do modelo de detecção de objetos e todas devem estar altas.
1. No canto superior direito da página, clique em **Teste Rápido** e, em seguida, na caixa **URL da Imagem**, digite `https://aka.ms/test-fruit` e clique no botão *teste rápido de imagem* (&#10132;).
1. Visualize a previsão gerada.

    ![Captura de tela do teste de detecção de objetos](../media/test-object-detection.png)

1. Feche a janela **Teste Rápido**.

## Usar o detector de objetos em um aplicativo cliente

Agora está tudo pronto para publicar seu modelo treinado e usá-lo em um aplicativo cliente.

### Publicar o modelo de detecção de objeto

1. No portal Visão Personalizada, na página **Desempenho**, clique em **&#128504; Publicar** para publicar o modelo treinado com as seguintes configurações:
    - **Nome do modelo**: `fruit-detector`
    - **Recurso de previsão**: *o recurso de **previsão** que você criou anteriormente que termina com "-Prediction" (<u>não</u> o recurso de treinamento)*.
1. No canto superior esquerdo da página **Configurações do Projeto**, clique no ícone *Galeria de Projetos* (&#128065;) para retornar à página inicial do portal Visão Personalizada, onde seu projeto agora está listado.
1. Na página inicial do portal Visão Personalizada, no canto superior direito, clique no ícone *configurações* (&#9881;) para exibir as configurações do serviço Visão Personalizada. Em seguida, em **Recursos**, localize seu recurso de *previsão* que termina com "-Prediction" (<u>não</u> o recurso de treinamento) para determinar seus valores de **Chave** e **ponto de extremidade** (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar o classificador de imagens de um aplicativo cliente

Agora que você publicou o modelo de classificação de imagem, pode usá-lo em um aplicativo cliente. Mais uma vez, você pode optar por usar **C#** ou **Python**.

1. Retorne à guia do navegador que contém o portal do Azure e o painel do Cloud Shell.
1. No Cloud Shell, execute os seguintes comandos para alternar para a pasta do aplicativo cliente e exibir os arquivos que ele contém:

    ```
   cd ../test-detector
   ls -a -l
    ```

    A pasta contém a configuração do aplicativo e os arquivos de código para seu aplicativo. Ele também contém o arquivo de imagem **produce.jpg** a seguir, que você usará para testar seu modelo.

    ![Imagem de algumas frutas.](../media/produce.jpg)

1. Instale o pacote do SDK da Visão Personalizada da IA do Azure para previsão e quaisquer outros pacotes necessários executando os seguintes comandos:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. Insira o seguinte comando para editar o arquivo de configuração do aplicativo:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. Atualize os valores de configuração para refletir o **Ponto de extremidade** e a **Chave** do seu recurso de Visão Personalizada para *<u>previsão</u>*, a **ID do Projeto** do projeto de detecção de objetos e o nome do seu modelo publicado (que deve ser *fruit-detector*). Salve suas alterações (*CTRL+S*) e feche o editor de código (*CTRL+Q*).

1. Na linha de comando do Cloud Shell, insira o seguinte comando para abrir o arquivo de código do aplicativo cliente:

    ```
   code test-detector.py
    ```

1. Examine o código, observando os seguintes detalhes:
    - Os namespaces do SDK de Visão Personalizada da IA do Azure são importados.
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionPredictionClient** autenticado.
    - O objeto de cliente de previsão é usado para obter previsões de detecção de objetos para a imagem **produce.jpg**, especificando a ID do projeto e o nome do modelo na solicitação. As regiões marcadas previstas são então desenhadas na imagem e o resultado é salvo como **output.jpg**.
1. Feche o editor de código e insira o seguinte comando para executar o programa:

    ```
   python test-detector.py
    ```

1. Examine a saída do programa, que lista cada objeto detectado na imagem.
1. Observe que um arquivo de imagem chamado **output.jpg** é gerado. Use o comando (específico do Azure Cloud Shell) **download** para baixá-lo:

    ```
   download output.jpg
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo. A imagem deve ser semelhante a esta:

    ![Uma imagem com os objetos detectados realçados.](../media/object-detection-output.jpg )

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais.

1. Abra o portal do Azure em `https://portal.azure.com`,e na barra de pesquisa superior, procure os recursos criados neste laboratório.

1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso. Como alternativa, você pode excluir todo o grupo de recursos para limpar todos os recursos ao mesmo tempo.
   
## Mais informações

Para obter mais informações sobre a detecção de objetos com o serviço Visão Personalizada, consulte a [documentação da Visão Personalizada](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
