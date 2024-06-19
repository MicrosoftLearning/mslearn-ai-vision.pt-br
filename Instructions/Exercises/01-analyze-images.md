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

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK de Visão de IA do Azure para analisar imagens.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/01-analyze-images** e expanda a pasta **C-Sharp** ou a pasta **Python** dependendo da sua preferência de linguagem.
2. Clique com o botão direito do mouse na pasta **image-analysis** e abra um terminal integrado. Em seguida, instale o pacote do SDK da Visão de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Observação**: se você for solicitado a instalar extensões de kit de desenvolvimento, você pode fechar a mensagem com segurança.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
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
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
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

2. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código especifica o caminho para um arquivo de imagem e, em seguida, passa o caminho da imagem para duas outras funções (**AnalyzeImage** e **BackgroundForeground**). Essas funções ainda não estão totalmente implementadas.

3. Na função **AnalyzeImage**, sob o comentário **Obter resultado com recursos especificados a serem recuperados**, adicione o seguinte código:

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
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
    
4. Na função **AnalyzeImage**, sob o comentário **Exibir resultados da análise**, adicione o seguinte código (incluindo os comentários que indicam onde você adicionará mais código posteriormente):

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

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
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Salve suas alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **images**, observando que, além da legenda da imagem, uma lista de tags sugeridas é exibida.

## Detectar e localizar objetos em uma imagem

A *detecção de objetos* é uma forma específica de pesquisa visual computacional na qual objetos individuais dentro de uma imagem são identificados e sua localização indicada por uma caixa delimitadora.

1. Na função **AnalyzeImage**, sob o comentário **Obter objetos na imagem**, adicione o seguinte código:

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
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
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

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
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
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
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
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
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("backgroundForeground.png", "wb") as file:
    file.write(image)
print('  Results saved in backgroundForeground.png \n')
```
    
2. Salve as alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **images**, abrindo o arquivo **background.png** gerado na mesma pasta que o arquivo de código para cada imagem.  Observe como o plano de fundo foi removido de cada uma das imagens.

Vamos agora gerar um fosco em primeiro plano para nossas imagens.

3. No arquivo de código, localize a função **BackgroundForeground** e, sob o comentário **Definir a versão e o modo da API**, altere a variável mode para `foregroundMatting`.

4. Salve as alterações e execute o programa uma vez para cada um dos arquivos de imagem na pasta **images**, abrindo o arquivo **background.png** gerado na mesma pasta que o arquivo de código para cada imagem.  Observe como um fosco de primeiro plano foi gerado para suas imagens.

## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.

2. Na barra de pesquisa superior, procure a *conta multisserviço dos serviços de IA do Azure* e selecione o recurso de conta multisserviço dos serviços de IA do Azure que você criou neste laboratório.

3. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Neste exercício, você explorou alguns dos recursos de análise e manipulação de imagem do serviço de Visão de IA do Azure. O serviço também inclui recursos para detectar objetos e pessoas, além de outras tarefas de pesquisa visual computacional.

Para saber mais sobre o serviço de **Visão de IA do Azure**, confira a [documentação da Visão de IA do Azure](https://learn.microsoft.com/azure/ai-services/computer-vision/).
