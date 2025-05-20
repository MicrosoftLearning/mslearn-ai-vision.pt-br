---
lab:
  title: Detectar objetos em imagens com a Visão Personalizada de IA do Azure
---

# Detectar objetos em imagens com a Visão Personalizada de IA do Azure

Neste exercício, você usa o serviço Visão Personalizada para treinar um modelo de *Detecção de objetos* capaz de detectar e localizar três classes de frutas (maçã, banana e laranja) em uma imagem.

## Clone o repositório para este curso

Se você já clonou o repositório de código **mslearn-ai-vision** para o ambiente em que está trabalhando neste laboratório, abra-o no Visual Studio Code, caso contrário, siga estas etapas para cloná-lo agora.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**.

## Criar recursos de Visão Personalizada

Se você já tiver recursos de **Visão Personalizada** para treinamento e previsão em sua assinatura do Azure, poderá usar estes ou uma conta multisserviço existente neste exercício. Ou use as instruções a seguir para criar.

> **Observação**: se você usar uma conta multisserviço, a chave e o ponto de extremidade serão os mesmos para o treinamento e a previsão.

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

Para treinar um modelo de detecção de objetos, você precisa criar um projeto de Visão Personalizada com base em seu recurso de treinamento. Para fazer isso, você usará o portal de Visão Personalizada.

1. Em uma nova guia do navegador, abra o portal de Visão Personalizada em `https://customvision.ai` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Crie um novo projeto com as seguintes configurações:
    - **Nome**: Detectar Frutas
    - **Descrição**: Detecção de objetos para frutas.
    - **Recurso**: *o recurso de Visão Personalizada criado anteriormente*
    - **Tipos de projeto**: detecção de objetos
    - **Domínios**: Geral
3. Aguarde a criação e abertura do projeto no navegador.

## Adicionar e marcar imagens

Para treinar um modelo de detecção de objeto, você precisa carregar imagens que contenham as classes que serão identificadas pelo modelo e marcá-las para indicar caixas delimitadoras para cada instância de objeto.

1. No Visual Studio Code, exiba as imagens de treinamento na pasta**Labfiles/03-object-detection/training-images** onde você clonou o repositório. Esta pasta contém imagens de frutas.
2. No portal de Visão Personalizada, no seu projeto de detecção de objetos, selecione **Adicionar imagens** e faça upload de todas as imagens da pasta extraída.
3. Depois de carregar as imagens, selecione a primeira para abri-la.
4. Mantenha o mouse sobre qualquer objeto na imagem até que uma região detectada automaticamente seja exibida, como na imagem abaixo. Em seguida, selecione o objeto e, se necessário, reorganize a região para cercá-lo.

    ![A região padrão de um objeto](../media/object-region.jpg)

    Como alternativa, você pode simplesmente arrastar o objeto para criar uma região.

5. Quando a região envolver o objeto, adicione uma nova marcação com o tipo de objeto apropriado (*maçã*, *banana* ou *laranja*), conforme mostrado aqui:

    ![Um objeto marcado em uma imagem](../media/object-tag.jpg)

6. Selecione e marque os outros objetos na imagem, redimensionando as regiões e adicionando novas marcações conforme o necessário.

    ![Dois objetos marcados em uma imagem](../media/object-tags.jpg)

7. Use o link **>** à direita para ir para a próxima imagem e marcar os objetos. Em seguida, continue trabalhando em toda a coleção de imagens, marcando cada maçã, banana e laranja.

8. Quando terminar de marcar a última imagem, feche o editor de **Detalhes da imagem**. Na página **Imagens de Treinamento**, em **Marcações**, selecione **Marcado** para ver todas as imagens marcadas:

![Imagens marcadas em um projeto](../media/tagged-images.jpg)

## Usar a API de treinamento para carregar imagens

Você pode usar a interface no portal Visão Personalizada para marcar suas imagens, mas muitas equipes de desenvolvimento de IA usam outras ferramentas que geram arquivos contendo informações sobre marcas e regiões de objetos em imagens. Em cenários como este, você pode usar a API de treinamento de Visão Personalizada para carregar imagens marcadas para o projeto.

> **Observação**: neste exercício, você pode optar por usar a API do SDK do **C#** ou **do Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. Clique no ícone *configurações* (&#9881;) no canto superior direito da página **Imagens de treinamento** no portal Visão Personalizada para exibir as configurações do projeto.
2. Em **Geral** (à esquerda), anote a **ID do projeto** que identifica exclusivamente este projeto.
3. À direita, em **Recursos**, observe que a chave e o ponto de extremidade são mostrados. Esses são os detalhes do recurso de *treinamento* (você também pode obter essas informações exibindo o recurso no portal do Azure).
4. No Visual Studio Code, na pasta **Labfiles/03-object-detection**, expanda a pasta **C-Sharp** ou **Python** dependendo da sua preferência de linguagem.
5. Clique com o botão direito do mouse na pasta **train-detector** e abra um terminal integrado. Em seguida, instale o pacote de treinamento de visão personalizada executando o comando apropriado para sua preferência de linguagem:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.1
```

6. Exiba o conteúdo da pasta **train-detector** e observe que ela contém um arquivo para definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

    Abra o arquivo de configuração e atualize os valores de configuração que ele contém para refletir o ponto de extremidade e a chave para seu recurso de *treinamento* de Visão Personalizada e a ID do projeto para o projeto de detecção de objetos criado anteriormente. Salve suas alterações.

7. Na pasta **train-detector**, abra **tagged-images.json** e examine o JSON que ele contém. O JSON define uma lista de imagens, cada uma contendo uma ou mais regiões marcadas. Cada região marcada inclui um nome de marca e as coordenadas superior e esquerda e as dimensões de largura e altura da caixa delimitadora que contém o objeto marcado.

    > **Observação**: as coordenadas e dimensões neste arquivo indicam pontos relativos na imagem. Por exemplo, um valor de *altura* de 0,7 indica uma caixa que é 70% da altura da imagem. Algumas ferramentas de marcação geram outros formatos de arquivo nos quais os valores de coordenadas e dimensões representam pixels, polegadas ou outras unidades de medidas.

8. Observe que a pasta **train-detector** contém uma subpasta na qual os arquivos de imagem referenciados no arquivo JSON são armazenados.

9. Observe que a pasta **train-detector** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: train-detector.py

    Abra o arquivo de código e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionTrainingClient** autenticado, que é usado com a ID do projeto para criar uma referência de **Projeto** para o seu projeto.
    - A função **Upload_Images** extrai as informações de região marcada no arquivo JSON e as usa para criar um lote de imagens com regiões, que ele carrega para o projeto.

10. Retorne o terminal integrado para a pasta **train-detector** e digite o seguinte comando para executar o programa:
    
    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python train-detector.py
    ```
    
11. Aguarde até que o programa finalize. Em seguida, retorne ao seu navegador e visualize a página **imagens de treinamento** do seu projeto no portal Visão Personalizada (atualize o navegador, se necessário).
12. Verifique se algumas novas imagens marcadas foram adicionadas ao projeto.

## Treinar e testar um modelo

Agora que você marcou as imagens em seu projeto, está tudo pronto para treinar um modelo. Você

1. No projeto Visão Personalizada, clique em **Treinar** para treinar um modelo de detecção de objeto usando as imagens marcadas. Selecione a opção **Treinamento Rápido**.
2. Aguarde a conclusão do treinamento (pode demorar cerca de dez minutos) e mapeie as métricas de desempenho de *Precisão,*, *Recall* e *mAP* – Essas métricas medem a precisão da previsão do modelo de classificação e devem ser todas altas.
3. No canto superior direito da página, clique em **Teste Rápido** e, na caixa **URL da Imagem**, insira `https://aka.ms/apple-orange` e veja a previsão gerada. Em seguida, feche a janela **Teste Rápido**.

## Publicar o modelo de detecção de objeto

Agora está tudo pronto para publicar seu modelo treinado e poder usá-lo em um aplicativo cliente.

1. No portal Visão Personalizada, na página **Desempenho**, clique em **&#128504; Publicar** para publicar o modelo treinado com as seguintes configurações:
    - **Nome do modelo**: detector de frutas
    - **Recurso de previsão**: *o recurso de **previsão** que você criou anteriormente que termina com "-Prediction" (<u>não</u> o recurso de treinamento)*.
2. No canto superior esquerdo da página **Configurações do Projeto**, clique no ícone *Galeria de Projetos* (&#128065;) para retornar à página inicial do portal Visão Personalizada, onde seu projeto agora está listado.
3. Na página inicial do portal Visão Personalizada, no canto superior direito, clique no ícone *configurações* (&#9881;) para exibir as configurações do serviço Visão Personalizada. Em seguida, em **Recursos**, localize seu recurso de *previsão* que termina com "-Prediction" (<u>não</u> o recurso de treinamento) para determinar seus valores de **Chave** e **ponto de extremidade** (você também pode obter essas informações exibindo o recurso no portal do Azure).

## Usar o classificador de imagens de um aplicativo cliente

Agora que você publicou o modelo de classificação de imagem, pode usá-lo em um aplicativo cliente. Mais uma vez, você pode optar por usar **C#** ou **Python**.

1. No Visual Studio Code, navegue até a pasta **Labfiles/03-object-detection** e, na pasta de idioma preferido (**C-Sharp** ou **Python**), expanda a pasta **test-detector**.
2. Clique com o botão direito do mouse na pasta **test-detector** e abra um terminal integrado. Em seguida, insira o seguinte comando específico do SDK para instalar o pacote de previsão de visão personalizada:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.1
```

> **Observação**: o pacote SDK do Python inclui pacotes de treinamento e previsão e pode já estar instalado.

3. Abra o arquivo de configuração do aplicativo cliente (*appsettings.json* para C# ou *.env* para Python) e atualize os valores de configuração que ele contém para refletir o ponto de extremidade e a chave para o recurso de *previsão* de Visão Personalizada, a ID do projeto para o projeto de detecção de objetos e o nome do modelo publicado (que deve ser *fruit-detector*). Salve suas alterações.
4. Abra o arquivo de código para seu aplicativo cliente (*Program.cs* para C#, *test-detector.py* para Python) e revise o código que ele contém, observando os seguintes detalhes:
    - Os namespaces do pacote instalado são importados
    - A função **Principal** recupera as definições de configuração e usa a chave e o ponto de extremidade para criar um **CustomVisionPredictionClient** autenticado.
    - O objeto de cliente de previsão é usado para obter previsões de detecção de objetos para a imagem **produce.jpg**, especificando a ID do projeto e o nome do modelo na solicitação. As regiões marcadas previstas são então desenhadas na imagem e o resultado é salvo como **output.jpg**.
5. Retorne ao terminal integrado da pasta **test-detector** e digite o seguinte comando para executar o programa:

**C#**

```
dotnet run
```

**Python**

```
python test-detector.py
```

6. Após a conclusão do programa, exiba o arquivo resultante **output.jpg** para ver os objetos detectados na imagem.

## Mais informações

Para obter mais informações sobre a detecção de objetos com o serviço Visão Personalizada, consulte a [documentação da Visão Personalizada](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
