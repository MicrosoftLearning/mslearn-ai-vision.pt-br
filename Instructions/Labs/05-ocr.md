---
lab:
  title: Ler texto em imagens
  module: Module 11 - Reading Text in Images and Documents
---

# Ler texto em imagens

OCR (reconhecimento óptico de caracteres) é um subconjunto da pesquisa visual computacional que lida com a leitura de texto em imagens e documentos. O serviço **Visão de IA do Azure** fornece uma API para leitura de texto, que você explorará neste exercício.

## Provisionar um recurso da Pesquisa Visual Computacional

Caso ainda não tenha um na sua assinatura, provisione um recurso da **Pesquisa Visual Computacional**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Selecione **Criar um recurso**.
1. Na barra de pesquisa, pesquise por *Pesquisa Visual Computacional*, selecione **Pesquisa Visual Computacional** e crie o recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se você estiver usando uma assinatura restrita, talvez não tenha permissão para criar um novo grupo de recursos; use o que foi fornecido)*
    - **Região**: *escolha entre Leste dos EUA, Oeste dos EUA, França Central, Coreia Central, Norte da Europa, Sudeste Asiático, Oeste da Europa ou Leste Asiático\**
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: F0 gratuito

    \*No momento, todos os recursos da Visão de IA do Azure 4.0 estão disponíveis apenas nessas regiões.

1. Marque as caixas de seleção necessárias e crie o recurso.
1. Aguarde a conclusão da implantação e veja os detalhes da implantação.
1. Quando o recurso tiver sido implantado, vá até ele e exiba sua página **Chaves e Ponto de Extremidade**. Você precisará do ponto de extremidade e de uma das chaves desta página no próximo procedimento.

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
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Preparação para usar o SDK da Visão de IA do Azure

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK da Visão de IA do Azure para leitura de texto.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. Navegue até a pasta que contém os arquivos de código do aplicativo para seu idioma preferido:  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
    ```

1. Instale o pacote do SDK da Visão de IA do Azure e as dependências necessárias executando os comandos apropriados para sua preferência de idioma:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. Usando o comando`ls`, você pode exibir o conteúdo da pasta **computer-vision**. Nela você encontrará um arquivo para definições de configuração:

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

1. No arquivo de código, atualize os valores de configuração que ele contém para refletir o **ponto de extremidade** e uma **chave** de autenticação para o recurso da Pesquisa Visual Computacional.
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Usar a Visão de IA do Azure para ler texto em uma imagem

Um dos recursos do **SDK da Visão de IA do Azure** é a leitura de texto de uma imagem. Neste exercício, você concluirá um aplicativo cliente implantado parcialmente que usa o SDK da Visão de IA do Azure para leitura de texto de uma imagem.

1. A pasta **read-text** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: read-text.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Em seguida, neste comentário, adicione o seguinte código de linguagem específico para importar os namespaces necessários para usar o SDK da Visão de IA do Azure:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

1. No arquivo de código para seu aplicativo cliente, na função **Principal** , o código para carregar as definições de configuração foi fornecido. Em seguida, localize o comentário **Autenticar cliente de Visão de IA do Azure**. Em seguida, neste comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto de cliente da Visão de IA do Azure:

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```
    
    **Python**
    
    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

1. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código especifica o caminho para um arquivo de imagem e, em seguida, passa o caminho da imagem para a função **GetTextRead**. Essa função ainda não está implementada totalmente.

1. Vamos adicionar algum código ao corpo da função **GetTextRead**. Encontre o comentário **Use a função Análise de imagem para leitura de texto na imagem**. Em seguida, sob este comentário, adicione o seguinte código específico de idioma, observando que os recursos visuais são especificados ao chamar a função `Analyze`:

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

1. No código que você acabou de adicionar na função **GetTextRead** e no comentário **Retornar o texto detectado na imagem**, adicione o código a seguir (este código imprime o texto da imagem no console e gera a imagem **text.jpg** que realça o texto da imagem):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Somente para arquivo de aplicativo em C#, uma função auxiliar ainda é necessária para desenhar um polígono. Abaixo do comentário **Helper method to draw a polygon given an array of SKPoints**, adicione o seguinte código:

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. Na função **Main**, examine o código que é executado se o usuário selecionar a opção **1** do menu. Esse código chama a função **GetTextRead**, passando o caminho para o arquivo da imagem *Lincoln.jpg*.

1. Salve as alterações e feche o editor de código.

1. Na barra de ferramentas do Cloud Shell, selecione **Carregar/Baixar arquivos** e, em seguida, **Baixar**. Na nova caixa de diálogo, insira o seguinte caminho de arquivo e selecione **Baixar**:

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. Abra a imagem **Lincoln.jpg** para exibi-la.

1. Depois de exibir a imagem que o código processa, insira o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando solicitado, digite **1** e observe a saída, que é o texto extraído da imagem.

1. Na pasta **read-text**, uma imagem de **text.jpg** foi criada. Você pode baixá-lo usando o caminho de arquivo `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` e observar como há um polígono em torno de cada *linha* de texto.

1. Reabra o arquivo de código e localize o comentário **Return the position bounding box around each line**. Sob o comentário, adicione o código a seguir:

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

1. Salve suas alterações e feche o editor de código e, em seguida, insira o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando solicitado, digite **1** e observe a saída, que deve ser cada linha de texto na imagem com sua respectiva posição na imagem.

1. Abra o arquivo de código novamente e localize o comentário **Return each word detected in the image and the position bounding box around each word with the confidence level of each word**. Sob o comentário, adicione o código a seguir:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Salve suas alterações e feche o editor de código e, em seguida, insira o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando solicitado, digite **1** e observe a saída, que deve ser cada palavra do texto na imagem com sua respectiva posição na imagem. Observe como o nível de confiança de cada palavra também é retornado.

1. Baixe a imagem **text.jpg** novamente e observe como há um polígono em torno de cada *palavra*.

## Usar o SDK da Visão de IA do Azure para leitura de texto manuscrito de uma imagem

No exercício anterior, você leu um texto bem definido de uma imagem, mas às vezes também pode querer ler texto de anotações manuscritas ou papéis. A boa notícia é que o **SDK da Visão de IA do Azure** também pode ler texto manuscrito com o mesmo código que você usou para leitura de texto bem definido. Usaremos o mesmo código do exercício anterior, mas desta vez usaremos uma imagem diferente.

1. Baixe **Note.jpg** usando o caminho de arquivo `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` para exibir a próxima imagem que seu código processará.

1. No arquivo de código do seu aplicativo, na função **Principal**, examine o código que é executado se o usuário selecionar a opção **2** do menu. Esse código chama a função **GetTextRead**, passando o caminho para o arquivo de imagem *Note.jpg*.

1. No terminal, digite o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando solicitado, digite **2** e observe a saída, que é o texto extraído da imagem da nota.

1. Baixe a imagem **text.jpg** novamente e observe como há um polígono em torno de cada *palavra* da nota.

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.

1. Na barra de pesquisa superior, pesquise por *Pesquisa Visual Computacional* e selecione o recurso de Pesquisa Visual Computacional que você criou neste laboratório.

1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para saber mais sobre o serviço de **Visão de IA do Azure** para leitura de texto, confira a [documentação da Visão de IA do Azure](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
