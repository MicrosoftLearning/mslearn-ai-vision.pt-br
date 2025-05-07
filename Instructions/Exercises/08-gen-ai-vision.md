---
lab:
  title: Desenvolver um aplicativo de chat habilitado para visão
  description: Saiba como usar a Fábrica de IA do Azure para criar um aplicativo de IA generativa que aceite entradas de imagem.
---

# Desenvolver um aplicativo de chat habilitado para visão

Neste exercício, você usará o modelo de IA generativa *Phi-4-multimodal-instruct* para gerar respostas para prompts que incluem imagens. Você desenvolverá um aplicativo que fornece assistência de IA com produtos frescos em uma mercearia usando a Fábrica de IA do Azure e o serviço de inferência de modelo da IA do Azure.

Este exercício levará aproximadamente **30** minutos.

## Criar um projeto do Azure AI Foundry

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir:

    ![Captura de tela do portal do Azure AI Foundry.](../media/ai-foundry-home.png)

2. Na home page, selecione **+Criar projeto**.
3. No assistente **Criar um projeto**, insira um nome de projeto adequado e, se um hub existente for sugerido, escolha a opção de criar um novo. Em seguida, examine os recursos do Azure que serão criados automaticamente para dar suporte ao hub e ao projeto.
4. Selecione **Personalizar** e especifique as seguintes configurações para o hub:
    - **Nome do hub**: *um nome válido para o seu hub*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: selecione uma das seguintes regiões\*:
        - Leste dos EUA
        - Leste dos EUA 2
        - Centro-Norte dos EUA
        - Centro-Sul dos Estados Unidos
        - Suécia Central
        - Oeste dos EUA
        - Oeste dos EUA 3
    - **Conectar os Serviços de IA do Azure ou o OpenAI do Azure**: *Criar um novo recurso de Serviços de IA*
    - **Conectar-se à Pesquisa de IA do Azure**: Ignorar a conexão

    > \* No momento em que este artigo foi escrito, o modelo *Phi-4-multimodal-instruct* da Microsoft que usaremos neste exercício está disponível nessas regiões. Você pode verificar a disponibilidade regional mais recente para modelos específicos na [documentação da Fábrica de IA do Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). No caso de um limite de cota regional ser atingido mais adiante no exercício, há a possibilidade de você precisar criar outro recurso em uma região diferente.

5. Clique em **Avançar** e revise a configuração. Em seguida, selecione **Criar** e aguarde a conclusão do processo.
6. Quando o projeto for criado, feche todas as dicas exibidas e examine a página do projeto no Portal da Fábrica de IA do Azure, que deve ser semelhante à imagem a seguir:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](../media/ai-foundry-project.png)

## Implantar um modelo multimodal

Agora você já pode implantar um modelo multimodal que aceite entrada baseada em imagem. Você pode escolher entre vários modelos, incluindo o modelo *gpt-4o* do OpenAI. Neste exercício, usaremos um modelo *Phi-4-multimodal-instruct* que aceita prompts que incluem imagens.

1. Na barra de ferramentas no canto superior direito da página do projeto da Fábrica de IA do Azure, use o ícone **Visualizar recursos** (**&#9215;**) para garantir que o recurso **Implantar modelos no serviço de inferência de modelo da IA do Azure** esteja habilitado. Esse recurso garante que a implantação do modelo esteja disponível para o serviço de inferência de IA do Azure, que você usará no código do aplicativo.
2. No painel à esquerda do seu projeto, na seção **Meus ativos**, selecione a página **Modelos + pontos de extremidade**.
3. Na página **Modelos + pontos extremidades**, na guia **Implantações de modelo**, no menu **+ Implantar modelo**, selecione **Implantar modelo base**.
4. Procure o modelo **Phi-4-multimodal-instruct** na lista e, em seguida, selecione-o e confirme-o.
5. Concorde com o contrato de licença, se solicitado, e implante o modelo com as seguintes configurações selecionando **Personalizar** nos detalhes da implantação:
    - **Nome da implantação**: *Um nome válido para sua implantação de modelo*
    - **Tipo de implantação**: padrão global
    - **Detalhes da implantação**: *use as configurações padrão*
6. Aguarde que o estado de provisionamento de implantação seja **Concluído**.

## Testar o modelo no playground

Agora que você tem uma implantação de modelo multimodal, pode testá-la com um prompt baseado em imagem no playground de chat.

1. No painel de navegação à esquerda, selecione a página **Playgrounds** e abra o playground **Chat**.
1. 1. Em uma nova guia do navegador, baixe [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg) em `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg` e salve-o em uma pasta no sistema de arquivos local.
1. Na página do playground de chat, no painel **Configuração**, verifique se a implantação do modelo **Phi-4-multimodal-instruct** está selecionada.
1. No painel principal da sessão de chat, abaixo da caixa de entrada de chat, use o botão anexar (**&#128206;**) para fazer upload do arquivo de imagem *mango.jpeg* e, em seguida, adicione o texto `What desserts could I make with this fruit?` e envie o prompt.

    ![Captura de tela do playground de chat com um prompt baseado em imagem.](../media/chat-playground-image.png)

1. Revise a resposta, que dará orientações relevantes de sobremesas que você pode fazer usando uma manga.

## Criar um aplicativo cliente

Agora que você implantou o modelo, pode usar a implantação em um aplicativo cliente.

> **Dica**: você pode optar por desenvolver sua solução usando Python ou Microsoft C# *(em breve)*. Siga as instruções na seção apropriada para o idioma escolhido.

### Preparar a configuração de aplicativo

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.
2. Na área **Detalhes do projeto**, observe a **Cadeia de conexão do projeto**. Você usará essa cadeia de conexão para se conectar ao seu projeto em um aplicativo cliente.
3. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

    Feche todas as notificações de boas-vindas para ver a home page do portal do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Você pode redimensionar ou maximizar esse painel para facilitar o trabalho.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

5. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):


    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

7. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    **Python**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/python
    ```

    **C#**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/c-sharp
    ```

8. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

9. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    O arquivo é aberto em um editor de código.

10. No arquivo de código, substitua o espaço reservado **your_project_connection_string** pela cadeia de conexão do seu projeto (copiada da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure) e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do modelo Phi-4-multimodal-instruct.
11. Após substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **Clique com o botão direito > Salvar** para salvar as alterações e, em seguida, use o comando **CTRL+Q** ou **Clique com o botão direito > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Escrever código para se conectar ao projeto e obter um cliente de chat para o modelo

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

2. No arquivo de código, observe as instruções existentes que foram adicionadas na parte superior do arquivo para importar os namespaces do SDK necessários. Em seguida, localize o comentário **Add references**, adicione o seguinte código para referenciar os namespaces nas bibliotecas que você instalou anteriormente:

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        ImageContentItem,
        ImageUrl,
   )
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. Na função **main**, no comentário **Get configuration settings**, observe que o código carrega a cadeia de conexão do projeto e os valores do nome de implantação do modelo que você definiu no arquivo de configuração.
4. Localize o comentário **Initialize the project client** e adicione o seguinte código para se conectar ao seu projeto da Fábrica de IA do Azure usando as credenciais do Azure com as quais você entrou:

    **Python**

    ```python
   # Initialize the project client
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. No comentário **Get a chat client**, adicione o seguinte código para criar um objeto cliente para conversar com o modelo:

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

### Escrever código para enviar um prompt de imagem baseado em URL

1. Observe que o código inclui um loop para permitir que o usuário insira um prompt até digitar "quit". Na seção de loop, localize o comentário **Get a response to image input** e adicione o seguinte código para enviar um prompt que inclua a seguinte imagem:

    ![Uma foto de uma manga.](../media/orange.jpeg)

    **Python**

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imageUrl = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(new Uri(imageUrl))
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código; não feche-o ainda.

3. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para executar o aplicativo:

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Quando solicitado, insira o seguinte prompt:

    ```
   Suggest some recipes that include this fruit
    ```

5. Revise a resposta. Em seguida, insira `quit` para sair do programa.

### Modificar o código para carregar um arquivo de imagem local

1. No editor de código do código do aplicativo, na seção de loop, localize o código adicionado anteriormente no comentário **Get a response to image input**. Em seguida, modifique o código da seguinte maneira para carregar esse arquivo de imagem local:

    ![Uma foto de uma pitaia.](../media/mystery-fruit.jpeg)

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

    # Read and encode the image file
    with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

    # Include the image file data in the prompt
    data_url = f"data:{mime_type};base64,{base64_encoded_data}"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);
    
   // Include the image file data in the prompt
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(bytes: binaryImage, mimeType: mimeType) 
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código. Você também pode fechar o editor de código (**CTRL+Q**), se desejar.

3. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para executar o aplicativo:

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Quando solicitado, insira o seguinte prompt:

    ```
   What is this fruit? What recipes could I use it in?
    ```

5. Revise a resposta. Em seguida, insira `quit` para sair do programa.

    > **Observação**: neste aplicativo simples, não implementamos lógica para reter o histórico de conversas; portanto, o modelo tratará cada prompt como uma nova solicitação sem contexto do prompt anterior.

## Explore mais: (Se o tempo permitir)

Você aprendeu a usar o SDK de Inferência de IA do Azure e um modelo multimodal para implementar um aplicativo de IA generativa que pode responder a prompts baseados em imagem. Se você tiver tempo, a seguir são algumas ideias para explorar mais.

### Usar um modelo multimodal diferente

Você usou um modelo *Phi-4-multimodal-instruct* para gerar uma resposta a um prompt baseado em imagem. Agora vamos tentar um modelo *gpt-4o* do OpenAI.

1. Na Fábrica de IA do Azure, implante um modelo **gpt-4o** em um ponto de extremidade de Inferência de Modelo da IA do Azure (talvez seja necessário criar um novo recurso em uma região diferente).
1. Atualize o arquivo de configuração de código do seu aplicativo (*.env* para Python e *appsettings.json* para C#) para especificar o nome do modelo gpt-4o.
1. Execute o aplicativo como antes, usando os mesmos prompts (você pode reverter para o código que usa uma imagem baseada em URL, se desejar).

### Usar APIs do OpenAI

O código usado neste exercício é baseado no SDK da Inferência de IA do Azure, que funciona com qualquer modelo implantado em um ponto de extremidade da Inferência de Modelo de IA do Azure. Ao usar um modelo do OpenAI, você também pode usar o SDK do OpenAI.

As instruções a seguir pressupõem que você tenha concluído este exercício e a tarefa adicional acima para implantar e testar um modelo **gpt-4o**.

1. Instale (ou atualize) os pacotes necessários para seu aplicativo:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai
    ```
    
    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

1. Atualize os namespaces em seu arquivo de código removendo as referências *azure.ai-inference*):

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   import openai
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. Modifique o código para obter um cliente de chat:

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient chatClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

1. Modifique o código para obter uma conclusão com base em um arquivo de imagem local

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = chat_client.chat.completions.create(
        model=model_deployment,
        messages=[
            { "role": "system", "content": system_message },
            { "role": "user", "content": [  
                { 
                    "type": "text", 
                    "text": prompt 
                },
                { 
                    "type": "image_url",
                    "image_url": {
                        "url": data_url
                    }
                }
            ] } 
        ]
   )
   completion = response.choices[0].message.content
   print(completion)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);

   // Include the image file data in the prompt
   List<ChatMessage> messages =
   [
        new SystemChatMessage(system_message),
        new UserChatMessage(
            ChatMessageContentPart.CreateTextPart(prompt),
            ChatMessageContentPart.CreateImagePart(binaryImage, mimeType)),
   ];

   ChatCompletion completion = chatClient.CompleteChat(messages);
   Console.WriteLine(completion.Content[0].Text);
    ```

1. Salve suas alterações e execute o aplicativo para testá-lo com os mesmos prompts que você usou anteriormente.

## Limpar

Se tiver terminado de explorar o portal da Fábrica de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
