---
lab:
  title: Detectar e analisar rostos
  module: 'Module 10 - Detecting, Analyzing, and Recognizing Faces'
---

# Detectar e analisar rostos

A capacidade de detectar e analisar rostos humanos é uma capacidade central da IA. Neste exercício, você conhecerá dois Serviços de IA do Azure que servem para trabalhar com rostos em imagens: o serviço de **Visão de IA do Azure** e o de **Detecção Facial**.

> **Observação**: a partir de 21 de junho de 2022, os recursos dos serviços de IA do Azure que retornam informações de identificação pessoal são restritos aos clientes que tiverem [acesso limitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Além disso, capacidades que inferem o estado emocional não estão mais disponíveis. Essas restrições podem afetar este exercício de laboratório. Estamos trabalhando para resolver isso, mas por enquanto é possível que você encontre erros ao seguir as etapas abaixo. Por isso, pedimos desculpas. Para mais detalhes sobre as alterações feitas pela Microsoft e o porquê, consulte [Investimentos de IA responsável e proteções para reconhecimento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clone o repositório para este curso

Se você ainda não tiver feito isso, clone o repositório de código para este curso:

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute um comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` para uma pasta local (não importa qual).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto outros arquivos adicionais são instalados para que haja suporte a projetos com código C# no repositório.

    > **Observação**: caso seja solicitado que você adicione os ativos necessários para compilar e depurar, selecione **Agora não**.

## Provisionar um recurso dos Serviços de IA do Azure

Caso ainda não tenha um na sua assinatura, será necessário provisionar um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, busque *serviços de IA do Azure*, selecione **Serviços de IA do Azure** e crie um recurso de conta multisserviço correspondente com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se estiver usando uma assinatura restrita, talvez você não tenha permissão para criar um grupo de recursos, então use o que foi fornecido)*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0
3. Marque as caixas de seleção necessárias e crie o recurso.
4. Aguarde a conclusão da implantação e veja os detalhes da implantação.
5. Quando o recurso tiver sido implantado, vá até ele e veja a página **Chaves e Ponto de Extremidade** correspondente. Você precisará do ponto de extremidade e de uma das chaves desta página no próximo procedimento.

## Preparação para usar o SDK da Visão de IA do Azure

Neste exercício, você concluirá um aplicativo cliente que foi parcialmente implementado e usa o SDK da Visão de IA do Azure para analisar rostos em uma imagem.

> **Observação**: você pode usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem da sua preferência.

1. No Visual Studio Code, no painel **Gerenciador**, navegue até **04-face** e abra a pasta **C-Sharp** ou **Python** dependendo da sua linguagem de preferência.
2. Clique com o botão direito do mouse na pasta **computer-vision** e abra um terminal integrado. Em seguida, instale o pacote do SDK da Visão de IA do Azure executando o comando apropriado para sua linguagem de preferência:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis --prerelease
    ```

    **Python**

    ```
    pip install azure-ai-vision
    ```
    
3. Veja o conteúdo da pasta **computer-vision** e observe que ela tem um arquivo de definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

4. Abra o arquivo de configuração e atualize os valores dele para refletir o **ponto de extremidade** e uma **chave** de autenticação para seu recurso de serviços de IA do Azure. Salve suas alterações.

5. Observe que a pasta **computer-vision** tem um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: detect-people.py

6. Abra o arquivo de código e, na parte superior, sob as referências de namespace, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de cada linguagem para importar os namespaces que você precisará para usar o SDK da Visão de IA do Azure:

    **C#**

    ```C#
    // import namespaces
    using Azure.AI.Vision.Common;
    using Azure.AI.Vision.ImageAnalysis;
    ```

    **Python**

    ```Python
    # import namespaces
    import azure.ai.vision as sdk
    ```

## Ver a imagem que analisará

Neste exercício, você usará o serviço de Visão de IA do Azure para analisar uma imagem de pessoas.

1. No Visual Studio Code, expanda a pasta **computer-vision** e a pasta **images** que está dentro da primeira.
2. Selecione a imagem **people.jpg** para visualizá-la.

## Detectar faces em uma imagem

Agora está tudo pronto para usar o SDK para chamar o serviço de Visão e detectar rostos em uma imagem.

1. No arquivo de código do seu aplicativo cliente (**Program.cs** ou **detect-people.py**), na função **Principal**, observe que foi fornecido o código para carregar as definições de configuração. Em seguida, localize o comentário **Autenticar cliente da Visão de IA do Azure**. Em seguida, embaixo deste comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto cliente da Visão de IA do Azure:

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

2. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código especifica o caminho para um arquivo de imagem e, em seguida, passa o caminho da imagem para uma função chamada **AnalyzeImage**. Esta função não foi totalmente implementada.

3. Na função **AnalyzeImage**, no comentário **Especificar recursos a serem recuperados (PEOPLE),** adicione o seguinte código:

    **C#**

    ```C#
    // Specify features to be retrieved (PEOPLE)
    Features =
        ImageAnalysisFeature.People
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (PEOPLE)
    analysis_options = sdk.ImageAnalysisOptions()
    
    features = analysis_options.features = (
        sdk.ImageAnalysisFeature.PEOPLE
    )    
    ```

4. Na função **AnalyzeImage**, sob o comentário **Obter análise da imagem**, adicione o seguinte código:

    **C#**

    ```C
    // Get image analysis
    using var imageSource = VisionSource.FromFile(imageFile);
    
    using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);
    
    var result = analyzer.Analyze();
    
    if (result.Reason == ImageAnalysisResultReason.Analyzed)
    {
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
                // Draw object bounding box if confidence > 50%
                if (person.Confidence > 0.5)
                {
                    // Draw object bounding box
                    var r = person.BoundingBox;
                    Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
                    graphics.DrawRectangle(pen, rect);
        
                    // Return the confidence of the person detected
                    Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
                }
            }
        
            // Save annotated image
            String output_file = "detected_people.jpg";
            image.Save(output_file);
            Console.WriteLine("  Results saved in " + output_file + "\n");
        }
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
                # Draw object bounding box if confidence > 50%
                if detected_people.confidence > 0.5:
                    # Draw object bounding box
                    r = detected_people.bounding_box
                    bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
                    draw.rectangle(bounding_box, outline=color, width=3)
            
                    # Return the confidence of the person detected
                    print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
                    
            # Save annotated image
            plt.imshow(image)
            plt.tight_layout(pad=0)
            outputfile = 'detected_people.jpg'
            fig.savefig(outputfile)
            print('  Results saved in', outputfile)
    
    else:
        error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
        print(" Analysis failed.")
        print("   Error reason: {}".format(error_details.reason))
        print("   Error code: {}".format(error_details.error_code))
        print("   Error message: {}".format(error_details.message))
    ```

5. Salve suas alterações e retorne ao terminal integrado para a pasta **computer-vision** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. Observe a saída, que deve indicar o número de rostos detectados.
7. Exiba o arquivo **detected_people.jpg** gerado na mesma pasta que o arquivo de código para ver os rostos observados. Nesse caso, seu código usou os atributos de rosto para rotular o local da parte superior esquerda da caixa e as coordenadas da caixa delimitadora para desenhar um retângulo ao redor de cada rosto.

## Prepare-se para usar o SDK de Detecção Facial

Embora o serviço **Visão de IA do Azure** ofereça detecção facial básica (além de vários outros recursos de análise de imagem), o serviço **Detecção facial** fornece funcionalidade mais abrangente para análise e reconhecimento facial.

1. No Visual Studio Code, no painel **Gerenciador**, navegue até a pasta **19-face** e expanda a pasta **C-Sharp** ou **Python**, dependendo da sua preferência de linguagem.
2. Clique com o botão direito do mouse na pasta **face-api** e abra um terminal integrado. Em seguida, instale o pacote do SDK de Detecção Facial executando o comando apropriado da sua preferência de idioma:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.8.0-preview.3
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.6.0
    ```
    
3. Exiba o conteúdo da pasta **face-api** e observe que ela contém um arquivo para as definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

4. Abra o arquivo de configuração e atualize os valores de configuração que ele contém para refletir o **ponto de extremidade** e a **chave**de autenticação do seu recurso de serviços de IA do Azure. Salve suas alterações.

5. Observe que a pasta **face-api** contém um arquivo de código do aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

6. Abra o arquivo de código e, na parte superior, sob as referências de namespace, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK da Visão:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. Na função **Principal**, observe que foi fornecido o código para carregar as definições de configuração. Em seguida, localize o comentário **Autenticar cliente de Detecção Facial**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código exibe um menu que permite chamar funções no código para explorar os recursos do serviço de Detecção Facial. Você implementará essas funções no restante deste exercício.

## Detectar e analisar rostos

Um dos recursos mais fundamentais do serviço de Detecção Facial é detectar rostos em uma imagem e determinar seus atributos, como postura da cabeça, desfoque, uso de óculos e assim por diante.

1. No arquivo de código do seu aplicativo, na função **Principal**, examine o código executado se o usuário selecionar a opção de menu **1**. Esse código chama a função **DetectFaces**, passando o caminho para um arquivo de imagem.
2. Localize a função **DetectFaces** no arquivo de código e, no comentário **Especificar atributos faciais a serem recuperados**, adicione o seguinte código:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    IList<FaceAttributeType> features = new FaceAttributeType[]
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. Na função **DetectFaces**, sob o código que você acabou de adicionar, localize o comentário **Identificar rostos** e adicione o seguinte código:

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count() > 0)
    {
        Console.WriteLine($"{detected_faces.Count()} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.White);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face number {faceCount}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face number {}'.format(face_count)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. Examine o código adicionado à função **DetectFaces**. Ele analisa um arquivo de imagem e detecta os rostos na imagem, incluindo atributos de oclusão, desfoque e uso de óculos. Os detalhes de cada rosto são exibidos, incluindo um identificador facial exclusivo que é atribuído a cada rosto, e a localização dos rostos é indicado na imagem usando uma caixa delimitadora.
5. Salve suas alterações e retorne ao terminal integrado para a pasta **face-api** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    *A saída C# pode exibir avisos sobre funções assíncronas agora que usam o operador **await**. Você pode ignorá-los.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Quando solicitado, digite **1** e observe a saída, que deve incluir o ID e os atributos de cada rosto detectado.
7. Exiba o arquivo **detected_faces.jpg** gerado na mesma pasta que o arquivo de código para ver os rostos observados.

## Mais informações

Existem vários recursos adicionais disponíveis no serviço de **Detecção Facial**; no entanto, seguindo o [Padrão de IA Responsável](https://aka.ms/aah91ff), eles são restritos por uma política de Acesso Limitado. Esses recursos incluem identificar, verificar e criar modelos de reconhecimento facial. Para saber mais e solicitar acesso, consulte o [Acesso Limitado de Serviços de IA do Azure](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Para saber mais sobre a detecção facial com o serviço da **Visão de IA do Azure**, confira a [documentação da Visão de IA do Azure](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Para saber mais sobre o serviço de **Detecção Facial**, confira a [documentação sobre a Detecção Facial](https://docs.microsoft.com/azure/cognitive-services/face/).
