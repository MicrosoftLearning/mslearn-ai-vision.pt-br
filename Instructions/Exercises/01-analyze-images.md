---
lab:
  title: Analisar imagens com a Visão de IA do Azure
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Analisar imagens com a Visão de IA do Azure

O Visão de IA do Azure é um recurso de inteligência artificial que permite que os sistemas de software interpretem a entrada visual analisando imagens. No Microsoft Azure, o serviço **Visão** de IA do Azure fornece modelos pré-criados para tarefas comuns de pesquisa visual computacional, incluindo análise de imagens para sugerir legendas e marcas, detecção de objetos comuns e pessoas. Você também pode usar o serviço de Visão de IA do Azure para remover o plano de fundo ou realçar a cobertura em primeiro plano.

## Clone o repositório para este curso

Se você ainda não clonou o repositório de código de **Visão de IA do Azure** para o ambiente em que está trabalhando neste laboratório, siga estas etapas para fazer isso. Caso contrário, abra a pasta clonada no Visual Studio Code.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute um comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` para uma pasta local (não importa qual pasta).
3. Quando o repositório tiver sido clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**. Se você for surpreendido com a mensagem *Detectado um projeto de função do Azure na pasta*, você pode fechar essa mensagem com segurança.

## Provisionar um recurso dos Serviços de IA do Azure

Caso ainda não tenha um na sua assinatura, provisione um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.
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

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK de Visão de IA do Azure para analisar imagens.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/01-analyze-images** e expanda a pasta **C-Sharp** ou a pasta **Python** dependendo da sua preferência de linguagem.
2. Clique com o botão direito do mouse na pasta **image-analysis** e abra um terminal integrado. Em seguida, instale o pacote do SDK da Visão de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis --prerelease
    ```

    > **Observação**: se você for solicitado a instalar extensões de kit de desenvolvimento, você pode fechar a mensagem com segurança.

    **Python**
    
    ```
    pip install azure-ai-vision
    ```
    
3. Exiba o conteúdo da pasta **image-analysis** e observe que ela contém um arquivo para definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

    Abra o arquivo de configuração e atualize os valores de configuração para refletir o **ponto de extremidade** e uma **chave** de autenticação para seu recurso de Serviços de IA do Azure. Salve suas alterações.
4. Observe que a pasta **image-analysis** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: image-analysis.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Visão de IA do Azure:

**C#**

```C#
// Import namespaces
using Azure.AI.Vision.Common;
using Azure.AI.Vision.ImageAnalysis;
```

**Python**

```Python
# import namespaces
import azure.ai.vision as sdk
```
    
## Veja as imagens que você vai analisar

Neste exercício, você usará o serviço de Visão de IA do Azure para analisar múltiplas imagens.

1. No Visual Studio Code, expanda a pasta **image-analysis** e **imagens** que ela contém.
2. Selecione cada um dos arquivos de imagem por vez para exibi-lo no Visual Studio Code.

## Analisar uma imagem para sugerir uma legenda

Agora você está pronto para usar o SDK para chamar o serviço de Visão e analisar uma imagem.

1. No arquivo de código do aplicativo cliente (**Program.cs** ou **image-analysis.py**), na função **Principal**, observe que o código para carregar as definições de configuração foi fornecido. Em seguida, localize o comentário **Autenticar cliente de Visão de IA do Azure**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto cliente de Visão de IA do Azure:

**C#**

```C#
// Authenticate Azure AI Vision client
var cvClient = new VisionServiceOptions(
    aiSvcEndpoint,
    new AzureKeyCredential(aiSvcKey));
```

**Python**

```Python
# Authenticate Azure AI Vision client
cv_client = sdk.VisionServiceOptions(ai_endpoint, ai_key)
```

2. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código especifica o caminho para um arquivo de imagem e, em seguida, passa o caminho da imagem para duas outras funções (**AnalyzeImage** e **BackgroundForeground**). Essas funções ainda não estão totalmente implementadas.

3. Na função **AnalyzeImage**, no comentário **Especificar recursos a serem recuperados**, adicione o seguinte código:

**C#**

```C#
// Specify features to be retrieved
Features =
    ImageAnalysisFeature.Caption
    | ImageAnalysisFeature.DenseCaptions
    | ImageAnalysisFeature.Objects
    | ImageAnalysisFeature.People
    | ImageAnalysisFeature.Text
    | ImageAnalysisFeature.Tags
```

**Python**

```Python
# Specify features to be retrieved
analysis_options = sdk.ImageAnalysisOptions()

features = analysis_options.features = (
    sdk.ImageAnalysisFeature.CAPTION |
    sdk.ImageAnalysisFeature.DENSE_CAPTIONS |
    sdk.ImageAnalysisFeature.TAGS |
    sdk.ImageAnalysisFeature.OBJECTS |
    sdk.ImageAnalysisFeature.PEOPLE
)
```
    
4. Na função **AnalyzeImage**, sob o comentário **Obter análise de imagem**, adicione o seguinte código (incluindo os comentários indicando onde você adicionará mais código mais tarde.):

**C#**

```C#
// Get image analysis
using var imageSource = VisionSource.FromFile(imageFile);

using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

var result = analyzer.Analyze();

if (result.Reason == ImageAnalysisResultReason.Analyzed)
{
    // get image captions
    if (result.Caption != null)
    {
        Console.WriteLine(" Caption:");
        Console.WriteLine($"   \"{result.Caption.Content}\", Confidence {result.Caption.Confidence:0.0000}");
    }

    //get image dense captions
    if (result.DenseCaptions != null)
    {
        Console.WriteLine(" Dense Captions:");
        foreach (var caption in result.DenseCaptions)
        {
            Console.WriteLine($"   \"{caption.Content}\", Confidence {caption.Confidence:0.0000}");
        }
        Console.WriteLine($"\n");
    }

    // Get image tags


    // Get objects in the image


    // Get people in the image

}
else
{
    var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
    Console.WriteLine(" Analysis failed.");
    Console.WriteLine($"   Error reason : {errorDetails.Reason}");
    Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
    Console.WriteLine($"   Error message: {errorDetails.Message}\n");
}
```

**Python**

```Python
# Get image analysis
image = sdk.VisionSource(image_file)

image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:
    # Get image captions
    if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.content, result.caption.confidence * 100))

    # Get image dense captions
    if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.content, caption.confidence * 100))

    # Get image tags


    # Get objects in the image


    # Get people in the image


else:
    error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
    print(" Analysis failed.")
    print("   Error reason: {}".format(error_details.reason))
    print("   Error code: {}".format(error_details.error_code))
    print("   Error message: {}".format(error_details.message))
```
    
5. Salve suas alterações e retorne ao terminal integrado para a pasta **image-analysis** e insira o seguinte comando para executar o programa com o argumento **images/street.jpg**:

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Observe a saída, que deve incluir uma legenda sugerida para a imagem **street.jpg**.
7. Execute o programa novamente, desta vez com o argumento **images/building.jpg** para ver a legenda que é gerada para a imagem **building.jpg**.
8. Repita a etapa anterior para gerar uma legenda para o arquivo **images/person.jpg**.

## Obter tags sugeridas para uma imagem

Às vezes, pode ser útil identificar *tags* relevantes que fornecem pistas sobre o conteúdo de uma imagem.

1. Na função **AnalyzeImage**, sob o comentário **Obter marcas de imagem** adicione o seguinte código:

**C#**

```C#
// Get image tags
if (result.Tags != null)
{
    Console.WriteLine($" Tags:");
    foreach (var tag in result.Tags)
    {
        Console.WriteLine($"   \"{tag.Name}\", Confidence {tag.Confidence:0.0000}");
    }
    Console.WriteLine($"\n");
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Salve suas alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **images**, observando que, além da legenda da imagem, uma lista de tags sugeridas é exibida.

## Detectar e localizar objetos em uma imagem

A *detecção de objetos* é uma forma específica de pesquisa visual computacional na qual objetos individuais dentro de uma imagem são identificados e sua localização indicada por uma caixa delimitadora.

1. Na função **AnalyzeImage**, sob o comentário **Obter objetos na imagem**, adicione o seguinte código:

**C#**

```C#
// Get objects in the image
if (result.Objects != null)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (var detectedObject in result.Objects)
    {
        Console.WriteLine($"   \"{detectedObject.Name}\", Confidence {detectedObject.Confidence:0.0000}");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Name,font,brush,r.X, r.Y);
    }
                    
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");   
}

```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_file)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.name, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. Salve suas alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **images**, observando todos os objetos que são detectados. Após cada execução, exiba o arquivo **objects.jpg** gerado na mesma pasta do arquivo de código para ver os objetos anotados.

## Detectar e localizar pessoas em uma imagem

A *detecção facial* é uma forma específica de pesquisa visual computacional na qual cada uma das pessoas dentro de uma imagem são identificadas e sua localização indicada por uma caixa delimitadora.

1. Na função **AnalyzeImage**, sob o comentário **Obter pessoas na imagem**, adicione o seguinte código:

**C#**

```C#
// Get people in the image
if (result.People != null)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (var person in result.People)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_file)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (Opcional) Remova o comentário do comando **Console.Writeline** na seção **Retornar a confiança da pessoa detectada** para revisar o nível de confiança retornado de que uma pessoa foi detectada em uma posição específica da imagem.

3. Salve suas alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **images**, observando todos os objetos que são detectados. Após cada execução, exiba o arquivo **objects.jpg** gerado na mesma pasta do arquivo de código para ver os objetos anotados.

> **Observação**: nas tarefas anteriores, você usou um único método para analisar a imagem e, em seguida, adicionou código para analisar e exibir os resultados. O SDK também fornece métodos individuais para sugerir legendas, identificar tags, detectar objetos e assim por diante – o que significa que você pode usar o método mais apropriado para retornar apenas as informações necessárias, reduzindo o conteúdo de dados que precisa ser retornado. Consulte a [documentação do SDK .NET](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) ou do [SDK Python](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision) para obter mais detalhes.

## Remover o plano de fundo ou gerar um fosco de primeiro plano de uma imagem

Em alguns casos, talvez seja necessário criar a remoção do plano de fundo de uma imagem ou talvez seja necessário criar um fosco de primeiro plano dessa imagem. Vamos começar com a remoção do plano de fundo.

1. No arquivo de código, localize a função **BackgroundForeground** e, sob o comentário **Remova o plano de fundo da imagem ou gere um fosco de primeiro plano**, adicione o seguinte código:

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($"\nRemove the background from the image or generate a foreground matte");

using var imageSource = VisionSource.FromFile(imageFile);

var analysisOptions = new ImageAnalysisOptions()
{
    // Set the image analysis segmentation mode to background or foreground
    SegmentationMode = ImageSegmentationMode.BackgroundRemoval
};

using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

var result = analyzer.Analyze();

// Remove the background or generate a foreground matte
if (result.Reason == ImageAnalysisResultReason.Analyzed)
{
    using var segmentationResult = result.SegmentationResult;

    var imageBuffer = segmentationResult.ImageBuffer;
    Console.WriteLine($"\n Segmentation result:");
    Console.WriteLine($"   Output image buffer size (bytes) = {imageBuffer.Length}");
    Console.WriteLine($"   Output image height = {segmentationResult.ImageHeight}");
    Console.WriteLine($"   Output image width = {segmentationResult.ImageWidth}");

    string outputImageFile = "newimage.jpg";
    using (var fs = new FileStream(outputImageFile, FileMode.Create))
    {
        fs.Write(imageBuffer.Span);
    }
    Console.WriteLine($"   File {outputImageFile} written to disk\n");
}
else
{
    var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
    Console.WriteLine(" Analysis failed.");
    Console.WriteLine($"   Error reason : {errorDetails.Reason}");
    Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
    Console.WriteLine($"   Error message: {errorDetails.Message}");
    Console.WriteLine(" Did you set the computer vision endpoint and key?\n");
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemove the background from the image or generate a foreground matte')

image = sdk.VisionSource(image_file)

analysis_options = sdk.ImageAnalysisOptions()

# Set the image analysis segmentation mode to background or foreground
analysis_options.segmentation_mode = sdk.ImageSegmentationMode.BACKGROUND_REMOVAL
    
image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:

    image_buffer = result.segmentation_result.image_buffer
    print(" Segmentation result:")
    print("   Output image buffer size (bytes) = {}".format(len(image_buffer)))
    print("   Output image height = {}".format(result.segmentation_result.image_height))
    print("   Output image width = {}".format(result.segmentation_result.image_width))

    output_image_file = "newimage.jpg"
    with open(output_image_file, 'wb') as binary_file:
        binary_file.write(image_buffer)
    print("   File {} written to disk".format(output_image_file))

else:

    error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
    print(" Analysis failed.")
    print("   Error reason: {}".format(error_details.reason))
    print("   Error code: {}".format(error_details.error_code))
    print("   Error message: {}".format(error_details.message))
    print(" Did you set the computer vision endpoint and key?")
```
    
2. Salve suas alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **imagens**, abrindo o arquivo **newimage.jpg** que é gerado na mesma pasta que o arquivo de código para cada imagem.  Observe como o plano de fundo foi removido de cada uma das imagens.

Vamos agora gerar um fosco em primeiro plano para nossas imagens.

3. No arquivo de código, localize a função **BackgroundForeground** e, sob o comentário **Defina o modo de segmentação de análise de imagem como plano de fundo ou primeiro plano**, substitua o código adicionado anteriormente pelo seguinte código:

**C#**

```C#
// Set the image analysis segmentation mode to background or foreground
SegmentationMode = ImageSegmentationMode.ForegroundMatting
```

**Python**

```Python
# Set the image analysis segmentation mode to background or foreground
analysis_options.segmentation_mode = sdk.ImageSegmentationMode.FOREGROUND_MATTING
```

4. Salve suas alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **imagens**, abrindo o arquivo **newimage.jpg** que é gerado na mesma pasta que o arquivo de código para cada imagem.  Observe como um fosco de primeiro plano foi gerado para suas imagens.

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.

2. Na barra de pesquisa superior, procure a *conta multisserviço dos serviços de IA do Azure* e selecione o recurso de conta multisserviço dos serviços de IA do Azure que você criou neste laboratório.

3. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Neste exercício, você explorou alguns dos recursos de análise e manipulação de imagem do serviço de Visão de IA do Azure. O serviço também inclui recursos para detectar objetos e pessoas, além de outras tarefas de pesquisa visual computacional.

Para saber mais sobre o serviço de **Visão de IA do Azure**, confira a [documentação da Visão de IA do Azure](https://learn.microsoft.com/azure/ai-services/computer-vision/).
