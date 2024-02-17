---
lab:
  title: Ler texto em imagens
  module: Module 11 - Reading Text in Images and Documents
---

# Ler texto em imagens

OCR (reconhecimento óptico de caracteres) é um subconjunto da pesquisa visual computacional que lida com a leitura de texto em imagens e documentos. O serviço **Visão de IA do Azure** fornece uma API para leitura de texto, que você explorará neste exercício.

## Clone o repositório para este curso

Se você ainda não tiver feito isso, deverá clonar o repositório de código para este curso:

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**. Se você for surpreendido com a mensagem *Detectado um projeto de função do Azure na pasta*, você pode fechar essa mensagem com segurança.

## Provisionar um recurso dos Serviços de IA do Azure

Caso ainda não tenha um na sua assinatura, provisione um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, pesquise *serviços de IA do Azure*, selecione **Serviços de IA do Azure** e crie um recurso de conta multisserviço dos serviços de IA do Azure com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se você estiver usando uma assinatura restrita, talvez não tenha permissão para criar um novo grupo de recursos; use o que foi fornecido)*
    - **Região**: *escolha entre Leste dos EUA, França Central, Coreia Central, Norte da Europa, Sudeste Asiático, Oeste da Europa, Oeste dos EUA ou Leste da Ásia\**
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0

    \*No momento, os recursos de Visão de IA do Azure 4.0 estão disponíveis apenas nessas regiões.

3. Marque as caixas de seleção necessárias e crie o recurso.
4. Aguarde a conclusão da implantação e veja os detalhes da implantação.
5. Quando o recurso tiver sido implantado, vá até ele e exiba sua página **Chaves e Ponto de Extremidade**. Você precisará do ponto de extremidade e de uma das chaves desta página no próximo procedimento.

## Preparação para usar o SDK da Visão de IA do Azure

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK da Visão de IA do Azure para leitura de texto.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas a seguir, execute as ações apropriadas para a sua linguagem preferida.

1. No Visual Studio Code, no painel **Explorer** navegue até a pasta **Labfiles\05-ocr** e expanda a pasta **C-Sharp** ou **Python** dependendo da sua linguagem de preferência.
2. Clique com o botão direito do mouse na pasta **read-text** e abra um terminal integrado. Em seguida, instale o pacote do SDK da Visão de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Observação**: se você for solicitado a instalar extensões de kit de desenvolvimento, você pode fechar a mensagem com segurança.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. Exiba o conteúdo da pasta **read text** e observe que ela contém um arquivo para definições de configuração:

    - **C#**: appsettings.json
    - **Python**: .env

    Abra o arquivo de configuração e atualize os valores de configuração para refletir o **ponto de extremidade** e uma **chave** de autenticação para seu recurso de Serviços de IA do Azure. Salve suas alterações.


## Usar a Visão de IA do Azure para ler texto em uma imagem

um dos recursos do **SDK da Visão de IA do Azure** é a leitura de texto de uma imagem. Neste exercício, você concluirá um aplicativo cliente implantado parcialmente que usa o SDK da Visão de IA do Azure para leitura de texto de uma imagem.

1. A pasta **read-text** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: read-text.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Em seguida, neste comentário, adicione o seguinte código de linguagem específico para importar os namespaces necessários para usar o SDK da Visão de IA do Azure:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. No arquivo de código para seu aplicativo cliente, na função **Principal** , o código para carregar as definições de configuração foi fornecido. Em seguida, localize o comentário **Autenticar cliente de Visão de IA do Azure**. Em seguida, neste comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto de cliente da Visão de IA do Azure:

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

3. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código especifica o caminho para um arquivo de imagem e, em seguida, passa o caminho da imagem para a função **GetTextRead**. Essa função ainda não está implementada totalmente.

4. Vamos adicionar algum código ao corpo da função **GetTextRead**. Encontre o comentário **Use a função Análise de imagem para leitura de texto na imagem**. Em seguida, sob este comentário, adicione o seguinte código específico de idioma, observando que os recursos visuais são especificados ao chamar a função `Analyze`:

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
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
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

5. No código que você acabou de adicionar na função **GetTextRead** e no comentário **Retornar o texto detectado na imagem**, adicione o código a seguir (este código imprime o texto da imagem no console e gera a imagem **text.jpg** que realça o texto da imagem):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return each line detected in the image and the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

6. Na pasta **read-text/images**, selecione **Lincoln.jpg** para exibir o arquivo que seu código processa.

7. No arquivo de código do seu aplicativo, na função **Principal**, examine o código executado se o usuário selecionar a opção **1** do menu. Esse código chama a função **GetTextRead**, passando o caminho para o arquivo da imagem *Lincoln.jpg*.

8. Salve suas alterações e retorne ao terminal integrado para a pasta **read-text** e digite o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. Quando solicitado, digite **1** e observe a saída, que é o texto extraído da imagem.

10. Na pasta **read-text**, selecione a imagem **text.jpg** e observe como há um polígono ao redor de cada *linha* de texto.

11. Retorne ao arquivo de código no Visual Studio Code e localize o comentário **Retornar a caixa delimitadora de posição em torno de cada linha**. Sob o comentário, adicione o código a seguir:

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

12. Salve suas alterações e retorne ao terminal integrado para a pasta **read-text** e digite o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. Quando solicitado, digite **1** e observe a saída, que deve ser cada linha de texto na imagem com sua respectiva posição na imagem.


14. Retorne ao arquivo de código no Visual Studio Code e localize o comentário **Retornar cada palavra detectada na imagem e a caixa delimitadora de posição ao redor de cada palavra com o nível de confiança de cada palavra**. Sob o comentário, adicione o código a seguir:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

15. Salve suas alterações e retorne ao terminal integrado para a pasta **read-text** e digite o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. Quando solicitado, digite **1** e observe a saída, que deve ser cada palavra do texto na imagem com sua respectiva posição na imagem. Observe como o nível de confiança de cada palavra também é retornado.

17. Na pasta de **read-text**, selecione a imagem **text.jpg** e observe como há um polígono ao redor de cada *palavra*.

## Usar o SDK da Visão de IA do Azure para leitura de texto manuscrito de uma imagem

No exercício anterior, você leu um texto bem definido de uma imagem, mas às vezes também pode querer ler texto de anotações manuscritas ou papéis. A boa notícia é que o **SDK da Visão de IA do Azure** também pode ler texto manuscrito com o mesmo código que você usou para leitura de texto bem definido. Usaremos o mesmo código do exercício anterior, mas desta vez usaremos uma imagem diferente.

1. Na pasta **read-text/images**, selecione **Note.jpg** para exibir o arquivo que seu código processa.

2. No arquivo de código do seu aplicativo, na função **Principal**, examine o código que é executado se o usuário selecionar a opção **2** do menu. Esse código chama a função **GetTextRead**, passando o caminho para o arquivo de imagem *Note.jpg*.

3. Do terminal integrado para a pasta **read- text**, digite o seguinte comando para executar o programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. Quando solicitado, digite **2** e observe a saída, que é o texto extraído da imagem da nota.

5. Na pasta **read-text**, selecione a imagem **text.jpg** e observe como há um polígono ao redor de cada *palavra* da nota.

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.

2. Na barra de pesquisa superior, procure a *conta multisserviço dos serviços de IA do Azure* e selecione o recurso de conta multisserviço dos serviços de IA do Azure que você criou neste laboratório

3. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para saber mais sobre o serviço de **Visão de IA do Azure** para leitura de texto, confira a [documentação da Visão de IA do Azure](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
