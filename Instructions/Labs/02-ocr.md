---
lab:
  title: Ler texto em imagens
  description: Use o OCR (reconhecimento óptico de caracteres) no serviço de Análise de Imagem da Visão de IA do Azure para localizar e extrair texto em imagens.
---

# Ler texto em imagens

OCR (reconhecimento óptico de caracteres) é um subconjunto da pesquisa visual computacional que lida com a leitura de texto em imagens e documentos. O serviço de Análise de Imagem da **Visão de IA do Azure** fornece uma API para leitura de texto, que você irá explorar neste exercício.

> **Observação**: Este exercício é baseado em software de SDK pré-lançamento, que pode estar sujeito a alterações. Quando necessário, usamos versões específicas de pacotes que podem não refletir as versões mais recentes disponíveis. Você pode experimentar algum comportamento inesperado, avisos ou erros.

Embora este exercício seja baseado no SDK de Análise da Visão do Azure para Python, você pode desenvolver aplicativos de visão usando vários SDKs específicos de linguagem, incluindo:

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

## Desenvolver um aplicativo de extração de texto com o SDK da Visão de IA do Azure

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK da Visão de IA do Azure para extrair texto de imagens.

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

1. Depois que o repositório tiver sido clonado, use o seguinte comando para navegar até os arquivos de código do aplicativo:

    ```
   cd mslearn-ai-vision/Labfiles/ocr/python/read-text
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

### Adicionar código para ler texto de uma imagem

1. Na linha de comando do Cloud Shell, insira o seguinte comando para abrir o arquivo de código do aplicativo cliente:

    ```
   code read-text.py
    ```

    > **Dica**: Talvez você queira maximizar o painel do Cloud Shell e mover a barra dividida entre o console de linha de comando e o editor de código para que você possa ver o código com mais facilidade.

1. No arquivo de código, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces necessários para usar o SDK do Visão de IA do Azure:

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Na função **Main**, o código para carregar as configurações e determinar o arquivo a ser analisado foi fornecido. Em seguida, localize o comentário **Authenticate Azure AI Vision client** e adicione o seguinte código específico da linguagem para criar e autenticar um objeto cliente de Análise de Imagem da Visão de IA do Azure:

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Na função **Main**, abaixo do código que você acabou de adicionar, localize o comentário **Read text in image** e adicione o seguinte código para usar o cliente de Análise de Imagem para ler o texto na imagem:

    ```python
   # Read text in image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print (f"\nReading text in {image_file}")

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ])
    ```

1. Localize o comentário **Print the text** e adicione o seguinte código (incluindo o comentário final) para imprimir as linhas de texto encontradas e chamar uma função para anotá-las na imagem (usando o **bounding_polygon** retornado para cada linha de texto):

    ```python
   # Print the text
   if result.read is not None:
        print("\nText:")
    
        for line in result.read.blocks[0].lines:
            print(f" {line.text}")        
        # Annotate the text in the image
        annotate_lines(image_file, result.read)

        # Find individual words in each line
        
    ```

1. Salve suas alterações (*CTRL+S*), mas mantenha o editor de código aberto caso precise corrigir qualquer erro de digitação.

1. Redimensione os painéis para conseguir ver mais do console e, em seguida, digite o seguinte comando para executar o programa:

    ```
   python read-text.py images/Lincoln.jpg
    ```

1. O programa lê o texto no arquivo de imagem especificado (*imagens/Lincoln.jpg*), que tem esta aparência:

    ![Fotografia de uma estátua de Abraham Lincoln.](../media/Lincoln.jpg)

1. Na pasta **read-text**, uma imagem **lines.jpg** foi criada. Use o comando (específico do Azure Cloud Shell) **download** para baixá-lo:

    ```
   download lines.jpg
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo. A imagem deve ser semelhante a esta:

    ![Uma imagem com o texto realçado.](../media/text.jpg)

1. Execute o programa novamente, desta vez especificando o parâmetro *images/Business-card.jpg* para extrair texto da seguinte imagem:

    ![Imagem de um cartão de visita escaneado.](../media/Business-card.jpg)

    ```
   python read-text.py images/Business-card.jpg
    ```

1. Baixe e visualize o arquivo **lines.jpg** resultante :

    ```
   download lines.jpg
    ```

1. Execute o programa mais uma vez, desta vez especificando o parâmetro *images/Note.jpg* para extrair texto desta imagem:

    ![Foto de uma lista de compras manuscritas.](../media/Note.jpg)

    ```
   python read-text.py images/Note.jpg
    ```

1. Baixe e visualize o arquivo **lines.jpg** resultante :

    ```
   download lines.jpg
    ```

### Adicionar código para retornar a posição de palavras individuais

1. Redimensione os painéis para que você possa ver mais do arquivo de código. Em seguida, localize o comentário **Find individual words in each line** e adicione o seguinte código (tomando cuidado para manter o nível correto de recuo):

    ```python
   # Find individual words in each line
   print ("\nIndividual words:")
   for line in result.read.blocks[0].lines:
        for word in line.words:
            print(f"  {word.text} (Confidence: {word.confidence:.2f}%)")
   # Annotate the words in the image
   annotate_words(image_file, result.read)
    ```

1. Salve suas alterações (*CTRL+S*). Em seguida, no painel da linha de comando, execute novamente o programa para extrair texto de *imagens/Lincoln.jpg*.
1. Observe a saída, que deve incluir cada palavra individual na imagem e a confiança associada à sua previsão.
1. Na pasta **read-text**, uma imagem **words.jpg** foi criada. Use o comando (específico do Azure Cloud Shell) **download** para baixá-lo e visualizá-lo:

    ```
   download words.jpg
    ```

1. Execute o programa novamente para *images/Business-card.jpg* e *images/Note.jpg*, visualizando o arquivo **words.jpg** gerado para cada imagem.

## Limpar os recursos

Se tiver terminado de explorar o Visão de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.

1. Na barra de pesquisa superior, pesquise por *Pesquisa Visual Computacional* e selecione o recurso de Pesquisa Visual Computacional que você criou neste laboratório.

1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.
