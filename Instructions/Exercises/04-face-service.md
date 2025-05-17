---
lab:
  title: Detectar e analisar rostos
  module: Module 4 - Detecting and Analyze Faces
---

# Detectar e analisar rostos

A capacidade de detectar e analisar rostos humanos é uma capacidade central de IA. Neste exercício, você explorará dois Serviços de IA do Azure que você pode usar para trabalhar com rostos em imagens: o serviço de **Visão de IA do Azure** e o serviço de **detecção facial**.

> **Importante**: Esse laboratório pode ser concluído sem solicitar acesso adicional a recursos restritos.

> **Observação**: a partir de 21 de junho de 2022, os recursos dos serviços de IA do Azure que retornam informações de identificação pessoal são restritos aos clientes que receberam [acesso ilimitado](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Além disso, capacidades que inferem o estado emocional não estão mais disponíveis. Para obter mais detalhes sobre as alterações feitas pela Microsoft e por quê – consulte [Investimentos em IA responsável e proteções para reconhecimento facial](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clone o repositório para este curso

Se você ainda não tiver feito isso, deverá clonar o repositório de código para este curso:

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**.

## Provisionar um recurso dos Serviços de IA do Azure

Caso ainda não tenha um na sua assinatura, provisione um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, pesquise *serviços de IA do Azure*, selecione **Serviços de IA do Azure** e crie um recurso de conta multisserviço dos serviços de IA do Azure com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se você estiver usando uma assinatura restrita, talvez não tenha permissão para criar um novo grupo de recursos; use o que foi fornecido)*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0
3. Marque as caixas de seleção necessárias e crie o recurso.
4. Aguarde a conclusão da implantação e veja os detalhes da implantação.
5. Quando o recurso tiver sido implantado, vá até ele e exiba sua página **Chaves e Ponto de Extremidade**. Você precisará do ponto de extremidade e de uma das chaves desta página no próximo procedimento.

## Preparação para usar o SDK da Visão de IA do Azure

Neste exercício, você concluirá um aplicativo cliente parcialmente implementado que usa o SDK de Visão de IA do Azure para analisar rostos em uma imagem.

> **Observação**: você pode optar por usar o SDK para **C#** ou **Python**. Nas etapas abaixo, execute as ações apropriadas para a linguagem de sua preferência.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/04-face** e expanda a pasta **C-Sharp** ou a pasta **Python** dependendo da sua preferência de idioma.
2. Clique com o botão direito do mouse na pasta **computer-vision** e abra um terminal integrado. Em seguida, instale o pacote do SDK da Visão de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```
    
3. Exiba o conteúdo da pasta **computer-vision** e observe que ela contém um arquivo para definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

4. Abra o arquivo de configuração e atualize os valores de configuração para refletir o **ponto de extremidade** e uma **chave** de autenticação para seu recurso de Serviços de IA do Azure. Salve suas alterações.

5. Observe que a pasta **computer-vision** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: detect-people.py

6. Abra o arquivo de código e, na parte superior, sob as referências de namespace, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Visão de IA do Azure:

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

## Veja a imagem que você vai analisar

Neste exercício, você usará o serviço de Visão de IA do Azure para analisar uma imagem com pessoas.

1. No Visual Studio Code, expanda a pasta **computer-vision** e a pasta **images** que ela contém.
2. Selecione a imagem **people.jpg** para visualizá-la.

## Detectar faces em uma imagem

Agora está tudo pronto para usar o SDK para chamar o serviço de Visão e detectar rostos em uma imagem.

1. No arquivo de código do seu aplicativo cliente (**Program.cs** ou **detect-people.py**), na função **Principal**, observe que foi fornecido o código para carregar as definições de configuração. Em seguida, localize o comentário **Autenticar cliente da Visão de IA do Azure**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto cliente de Visão de IA do Azure:

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
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

2. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código especifica o caminho para um arquivo de imagem e, em seguida, passa o caminho da imagem para uma função chamada **AnalyzeImage**. Esta função não foi totalmente implementada.

3. Na função **AnalyzeImage**, sob o comentário **Obter resultado com recursos especificados a serem recuperados (PESSOAS)**, adicione o seguinte código:

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

4. Na função **AnalyzeImage**, sob o comentário **Desenhar caixa delimitadora ao redor das pessoas detecadas**, adicione o seguinte código:

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
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
7. Exiba o arquivo **people.jpg** gerado na mesma pasta que o arquivo de código para ver os rostos anotados. Nesse caso, seu código usou os atributos de rosto para rotular o local da parte superior esquerda da caixa e as coordenadas da caixa delimitadora para desenhar um retângulo ao redor de cada rosto.

Se você quiser ver a pontuação de confiança de todas as pessoas detectadas pelo serviço, poderá remover a marca de comentário da linha de código abaixo do comentário `Return the confidence of the person detected` e executar novamente o código.

## Prepare-se para usar o SDK de Detecção Facial

Enquanto o serviço de **Visão de IA do Azure** oferece detecção facial básica (juntamente com muitos outros recursos de análise de imagem), o serviço de **Detecção Facial** fornece funcionalidade mais abrangente para análise e reconhecimento facial.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/04-face** e expanda a pasta **C-Sharp** ou a pasta **Python** dependendo da sua preferência de idioma.
2. Clique com o botão direito do mouse na pasta **face-api** e abra um terminal integrado. Em seguida, instale o pacote SDK de Detecção Facial executando o comando apropriado para sua preferência de linguagem:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
3. Exiba o conteúdo da pasta **face-api** e observe que ela contém um arquivo para definições de configuração:
    - **C#**: appsettings.json
    - **Python**: .env

4. Abra o arquivo de configuração e atualize os valores de configuração para refletir o **ponto de extremidade** e uma **chave** de autenticação para seu recurso de Serviços de IA do Azure. Salve suas alterações.

5. Observe que a pasta **face-api** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

6. Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK da Visão:

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

7. Na função **Principal**, observe que o código para carregar as definições de configuração foi fornecido. Em seguida, localize o comentário **Autenticar cliente de detecção facial**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para criar e autenticar um objeto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

8. Na função **Principal**, sob o código que você acabou de adicionar, observe que o código exibe um menu que permite chamar funções em seu código para explorar os recursos do serviço de detecção facial. Você implementará essas funções no restante deste exercício.

## Detectar e analisar rostos

Um dos recursos mais fundamentais do serviço de detecção facial é detectar rostos em uma imagem e determinar seus atributos, como postura da cabeça, desfoque, presença de máscara e assim por diante.

1. No arquivo de código do seu aplicativo, na função **Principal**, examine o código executado se o usuário selecionar a opção **1** do menu. Esse código chama a função **DetectFaces**, passando o caminho para um arquivo de imagem.
2. Localize a função **DetectFaces** no arquivo de código e, no comentário **Especificar recursos faciais a serem recuperados**, adicione o seguinte código:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

3. Na função **DetectFaces**, sob o código que você acabou de adicionar, localize o comentário **Obter faces** e adicione o seguinte código:

**C#**

```C#
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var response = await faceClient.DetectAsync(
        BinaryData.FromStream(imageData),
        FaceDetectionModel.Detection03,
        FaceRecognitionModel.Recognition04,
        returnFaceId: false,
        returnFaceAttributes: features);
    IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

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
            Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
            Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
            Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

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
    detected_faces = face_client.detect(
        image_content=image_data.read(),
        detection_model=FaceDetectionModel.DETECTION03,
        recognition_model=FaceRecognitionModel.RECOGNITION04,
        return_face_id=False,
        return_face_attributes=features,
    )

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

            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
            print(' - Mask: {}'.format(face.face_attributes.mask.type))

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

4. Examine o código adicionado à função **DetectFaces**. Ele analisa um arquivo de imagem e detecta qualquer rosto que tiver, incluindo atributos para postura da cabeça, desfoque e presença de máscara. Os detalhes de cada face são exibidos, incluindo um identificador facial exclusivo que é atribuído a cada face; e a localização das faces é indicada na imagem usando uma caixa delimitadora.
5. Salve suas alterações e retorne ao terminal integrado para a pasta **face-api** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    *A saída C# pode exibir avisos sobre funções assíncronas qie não estão usando o operador **await**. Você pode ignorá-los.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Quando solicitado, digite **1** e observe a saída, que deve incluir o ID e os atributos de cada face detectada.
7. Exiba o arquivo **detected_faces.jpg** gerado na mesma pasta que o arquivo de código para ver as faces anotadas.

## Mais informações

Existem vários recursos adicionais disponíveis no serviço de **Detecção facial**, mas seguindo o [Padrão de Uso Responsável de IA](https://aka.ms/aah91ff), eles são restritos por uma política de acesso limitado. Esses recursos incluem identificar, verificar e criar modelos de reconhecimento facial. Para saber mais e solicitar acesso, consulte o [Acesso limitado para serviços de IA do Azure](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Para saber mais sobre a detecção facial com o serviço da **Visão de IA do Azure**, confira a [documentação da Visão de IA do Azure](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Para saber mais sobre o serviço de **Detecção Facial**, confira a [documentação sobre a detecção facial](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity).
