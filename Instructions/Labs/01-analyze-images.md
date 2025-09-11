---
lab:
  title: Analisar imagens
  description: 'Use a Análise de Imagem da Visão de IA do Azure para analisar imagens, sugerir legendas e marcas e detectar objetos e pessoas.'
---

# Analisar imagens

O Visão de IA do Azure é um recurso de inteligência artificial que permite que os sistemas de software interpretem a entrada visual analisando imagens. No Microsoft Azure, o serviço **Visão** de IA do Azure fornece modelos pré-criados para tarefas comuns de pesquisa visual computacional, incluindo análise de imagens para sugerir legendas e marcas, detecção de objetos comuns e pessoas. Você também pode usar o serviço de Visão de IA do Azure para remover o plano de fundo ou realçar a cobertura em primeiro plano.

> **Observação**: Este exercício é baseado em software de SDK pré-lançamento, que pode estar sujeito a alterações. Quando necessário, usamos versões específicas de pacotes que podem não refletir as versões mais recentes disponíveis. Você pode experimentar algum comportamento inesperado, avisos ou erros.

Embora este exercício seja baseado no SDK do Visão do Azure para Python, você pode desenvolver aplicativos de Visão usando vários SDKs específicos de linguagem, incluindo:

* [Análise de Visão de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Análise de Visão de IA do Azure para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Análise de Visão de IA do Azure para Java](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

Este exercício levará aproximadamente **30** minutos.

## Provisionar um recurso da Visão de IA do Azure

Caso ainda não tenha um na sua assinatura, provisione um recurso de Visão de IA do Azure.

> **Observação**: Neste exercício, você usará um recurso autônomo de **Pesquisa Visual Computacional**. Você também pode usar os serviços de Visão de IA do Azure em um recurso multisserviço dos *Serviços de IA do Azure*, diretamente ou em um projeto da *Fábrica de IA do Azure*.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre usando suas credenciais do Azure. Feche todas as mensagens de boas-vindas ou dicas exibidas.
1. Selecione **Criar um recurso**.
1. Na barra de pesquisa, pesquise `Computer Vision`, selecione **Pesquisa Visual Computacional** e crie o recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *Escolha entre **Leste dos EUA**, **Oeste dos EUA**, **França Central**, **Coreia Central**, **Norte da Europa**, **Sudeste da Ásia**, **Oeste da Europa** ou **Leste da Ásia**\**
    - **Nome**: *Um nome válido para o recurso de Pesquisa Visual Computacional*
    - **Tipo de preço**: F0 gratuito

    \*No momento, todos os recursos da Visão de IA do Azure 4.0 estão disponíveis apenas nessas regiões.

1. Marque as caixas de seleção necessárias e crie o recurso.
1. Aguarde a conclusão da implantação e veja os detalhes da implantação.
1. Quando o recurso tiver sido implantado, vá até ele e, no nó de **Gerenciamento de recursos** no painel de navegação, exiba sua página **Chaves e Ponto de Extremidade**. Você precisará do ponto de extremidade e de uma das chaves desta página no próximo procedimento.

## Desenvolver um aplicativo de análise de imagens com o SDK do Visão de IA do Azure

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK de Visão de IA do Azure para analisar imagens.

### Preparar a configuração de aplicativo

1. No portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

    > **Observação**: Se o portal solicitar que você selecione um armazenamento para persistir seus arquivos, escolha **Nenhuma conta de armazenamento necessária**, selecione a assinatura que você está usando e pressione **Aplicar**.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. Redimensione o painel do Cloud Shell para que você ainda possa ver a página **Chaves e Ponto de Extremidade** do seu recurso de Pesquisa Visual Computacional.

    > **Dica**" Você pode redimensionar o painel arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Dica**: ao colar comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Depois que o repositório tiver sido clonado, use o seguinte comando para navegar e exibir a pasta que contém os arquivos de código do aplicativo:   

    ```
   cd mslearn-ai-vision/Labfiles/analyze-images/python/image-analysis
   ls -a -l
    ```

    A pasta contém a configuração do aplicativo e os arquivos de código para seu aplicativo. Ele também contém uma subpasta **/images**, que contém alguns arquivos de imagem para seu aplicativo analisar.
    
1. Instale o pacote de SDK do Visão de IA do Azure e outros pacotes necessários executando os seguintes comandos:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. Insira o seguinte comando para editar o arquivo de configuração do aplicativo:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, atualize os valores de configuração para refletir o **ponto de extremidade** e a **chave** de autenticação do seu recurso de Pesquisa Visual Computacional (copiados da página **Chaves e Ponto de Extremidade** no portal do Azure).
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Adicionar código para sugerir uma legenda

1. Na linha de comando do Cloud Shell, insira o seguinte comando para abrir o arquivo de código do aplicativo cliente:

    ```
   code image-analysis.py
    ```

    > **Dica**: Talvez você queira maximizar o painel do Cloud Shell e mover a barra dividida entre o console de linha de comando e o editor de código para que você possa ver o código com mais facilidade.

1. No arquivo de código, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces necessários para usar o SDK do Visão de IA do Azure:

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Na função **Main**, observe que o código para carregar as configurações e determinar o arquivo de imagem a ser analisado foi fornecido. Em seguida, localize o comentário **Authenticate Azure AI Vision client** e adicione o seguinte código para criar e autenticar um objeto cliente do Visão de IA do Azure (certifique-se de manter os níveis corretos de recuo):

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Na função **Main**, abaixo do código que você acabou de adicionar, localize o comentário **Analyze image** e adicione o seguinte código:

    ```python
   # Analyze image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print(f'\nAnalyzing {image_file}\n')

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.DENSE_CAPTIONS,
            VisualFeatures.TAGS,
            VisualFeatures.OBJECTS,
            VisualFeatures.PEOPLE],
   )
    ```

1. Localize o comentário **Get image captions**, adicione o seguinte código para exibir legendas de imagem e legendas densas:

    ```python
   # Get image captions
   if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))
    
   if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions.list:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))
    ```

1. Salve suas alterações (*CTRL+S*) e redimensione os painéis para que você consiga ver claramente o console de linha de comando enquanto mantém o editor de código aberto. Em seguida, insira o seguinte comando para executar o programa com o argumento **images/street.jpg**:

    ```
   python image-analysis.py images/street.jpg
    ```

1. Observe a saída, que deve incluir uma legenda sugerida para a imagem **street.jpg**, que tem esta aparência:

    ![Uma foto de uma rua movimentada.](../media/street.jpg)

1. Execute o programa novamente, desta vez com o argumento **images/building.jpg**, para ver a legenda gerada para a imagem **building.jpg**, que fica assim:

    ![Uma foto de um prédio.](../media/building.jpg)

1. Repita a etapa anterior para gerar uma legenda para o arquivo **images/person.jpg**, que tem esta aparência:

    ![Uma foto de uma pessoa.](../media/person.jpg)

### Adicionar código para gerar marcas sugeridas

Às vezes, pode ser útil identificar *tags* relevantes que fornecem pistas sobre o conteúdo de uma imagem.

1. No editor de código, na função **AnalyzeImage**, localize o comentário **Get image tags** e adicione o seguinte código:

    ```python
   # Get image tags
   if result.tags is not None:
        print("\nTags:")
        for tag in result.tags.list:
            print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
    ```

1. Salve suas alterações (*CTRL+S*) e execute o programa com o argumento **images/street.jpg**, observando que, além da legenda da imagem, uma lista de marcas sugeridas é exibida.
1. Execute novamente o programa para os arquivos **images/building.jpg** e **images/person.jpg**.

### Adicionar código para detectar e localizar objetos

1. No editor de código, na função **AnalyzeImage**, localize o comentário **Get objects in the image** e adicione o seguinte código para listar os objetos detectados na imagem e chamar a função fornecida para anotar a imagem com os objetos detectados:

    ```python
   # Get objects in the image
   if result.objects is not None:
        print("\nObjects in image:")
        for detected_object in result.objects.list:
            # Print object tag and confidence
            print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        # Annotate objects in the image
        show_objects(image_file, result.objects.list)
    ```

1. Salve suas alterações (*CTRL+S*) e execute o programa com o argumento **images/street.jpg**, observando que, além da legenda da imagem e das marcas sugeridas, um arquivo chamado **objects.jpg** foi gerado.
1. Use o comando (específico do Azure Cloud Shell) **download** para baixar o arquivo **objects.jpg**:

    ```
   download objects.jpg
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo. A imagem deve ser semelhante a esta:

    ![Uma imagem com caixas com marco de delimitação dos objetos.](../media/objects.jpg)

1. Execute o programa novamente para os arquivos **images/building.jpg** e **images/person.jpg**, baixando o arquivo objects.jpg gerado após cada execução.

### Adicionar código para detectar e localizar pessoas

1. No editor de código, na função **AnalyzeImage**, localize o comentário **Get people in the image** e adicione o seguinte código para listar quaisquer pessoas detectadas com nível de confiança de 20% ou mais e chamar a função fornecida para anotá-las na imagem:

    ```Python
   # Get people in the image
   if result.people is not None:
        print("\nPeople in image:")

        for detected_person in result.people.list:
            if detected_person.confidence > 0.2:
                # Print location and confidence of each person detected
                print(" {} (confidence: {:.2f}%)".format(detected_person.bounding_box, detected_person.confidence * 100))
        # Annotate people in the image
        show_people(image_file, result.people.list)
    ```

1. Salve suas alterações (*CTRL+S*) e execute o programa com o argumento **images/street.jpg**, observando que, além da legenda da imagem, das marcas sugeridas e do arquivo objects.jpg, é gerada uma lista de localizações de pessoas e um arquivo chamado **people.jpg**.

1. Use o comando (específico do Azure Cloud Shell) **download** para baixar o arquivo **objects.jpg**:

    ```
   download people.jpg
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo. A imagem deve ser semelhante a esta:

    ![Uma imagem com caixas com marco de delimitação para pessoas detectadas.](../media/people.jpg)

1. Execute o programa novamente para os arquivos **images/building.jpg** e **images/person.jpg**, baixando o arquivo people.jpg gerado após cada execução.

   > **Dica:** Se você notar que as caixas com marco de delimitação retornadas pelo modelo não fazem sentido, verifique a pontuação de confiança no JSON e tente aumentar o filtro de pontuação de confiança no seu aplicativo.

## Limpar os recursos

Se tiver terminado de explorar o Visão de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.

1. Na barra de pesquisa superior, pesquise por *Pesquisa Visual Computacional* e selecione o recurso de Pesquisa Visual Computacional que você criou neste laboratório.

1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

