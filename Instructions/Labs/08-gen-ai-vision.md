---
lab:
  title: Desenvolver um aplicativo de chat habilitado para visão
  description: Use a Fábrica de IA do Azure para criar um aplicativo de IA generativa que aceite entradas de imagem.
---

# Desenvolver um aplicativo de chat habilitado para visão

Neste exercício, você usará o modelo de IA generativa *Phi-4-multimodal-instruct* para gerar respostas para prompts que incluem imagens. Você desenvolverá um aplicativo que fornece assistência de IA com produtos frescos em uma mercearia usando a Fábrica de IA do Azure e o serviço de inferência de modelo da IA do Azure.

> **Observação**: Este exercício é baseado em software de SDK pré-lançamento, que pode estar sujeito a alterações. Quando necessário, usamos versões específicas de pacotes que podem não refletir as versões mais recentes disponíveis. Você pode experimentar algum comportamento inesperado, avisos ou erros.

Embora este exercício seja baseado no SDK do Python da Fábrica de IA do Azure, você pode desenvolver aplicativos de chat de IA usando vários SDKs específicos de linguagem; incluindo:

- [Projetos de IA do Azure para Python](https://pypi.org/project/azure-ai-projects)
- [Projetos de IA do Azure para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Projetos de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure/ai-projects)

Este exercício levará aproximadamente **30** minutos.

## Abra o portal do Azure AI Foundry

Vamos começar entrando no portal da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela do portal do Azure AI Foundry.](./media/ai-foundry-home.png)

1. Revise as informações na home page.

## Escolher um modelo para iniciar um projeto

Um *projeto* da IA do Azure fornecerá um workspace colaborativo para desenvolvimento de IA. Vamos começar escolhendo um modelo com o qual queremos trabalhar e criando um projeto para usá-lo.

> **Observação**: os projetos da Fábrica de IA podem ser baseados em um recurso da *Fábrica de IA do Azure*, que fornece acesso a modelos de IA (incluindo o OpenAI do Azure), aos Serviços de IA do Azure e outros recursos para desenvolver agentes de IA e soluções de chat. Como alternativa, os projetos podem ser baseados em recursos do *hub de IA*, que incluem conexões com recursos do Azure para armazenamento seguro, computação e ferramentas especializadas. Os projetos baseados na Fábrica de IA do Azure são ótimos para desenvolvedores que desejam gerenciar recursos para o desenvolvimento de aplicativos de chat ou agente de IA. Os projetos baseados em hub de IA são mais adequados para equipes de desenvolvimento empresarial que trabalham em soluções de IA complexas.

1. Na home page, na seção **Explorar modelos e recursos**, pesquise pelo modelo `Phi-4-multimodal-instruct`, que usaremos em nosso projeto.

1. Nos resultados da pesquisa, selecione o modelo **Phi-4-multimodal-instruct** para ver os detalhes e, na parte superior da página do modelo, clique em **Usar este modelo**.

1. Quando solicitado a criar um projeto, insira um nome válido para o projeto e expanda **Opções avançadas**.

1. Selecione **Personalizar** e especifique as seguintes configurações para o hub:
    - **Recurso da Fábrica de IA do Azure**: *um nome válido para o recurso da Fábrica de IA do Azure*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: *selecione qualquer **AI Foundry recomendado***\*

    > \* Alguns recursos da IA do Azure são restritos por cotas de modelo regional. Caso um limite de cota seja excedido posteriormente no exercício, é possível que você precise criar outro recurso em uma região diferente.

1. Clique em **Criar** e aguarde a criação do projeto, incluindo a implantação do modelo Phi-4-multimodal-instruct selecionado.

    > Observação: Dependendo da seleção do modelo, você pode receber solicitações adicionais durante o processo de criação do projeto. Concorde com os termos e finalize a implantação.

1. Quando o projeto for criado, seu modelo será exibido na página **Modelos + pontos de extremidade**:

    ![Captura de tela da página de implantação de modelo.](./media/ai-foundry-model-deployment.png)

## Testar o modelo no playground

Agora você pode testar sua implantação de modelo multimodal com um prompt baseado em imagem no playground de chat.

1. Selecione **Abrir no playground** na página de implantação do modelo.

1. Em uma nova guia do navegador, baixe [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg) em `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg` e salve-o em uma pasta no sistema de arquivos local.

1. Na página do playground de chat, no painel **Configuração**, verifique se a implantação do modelo **Phi-4-multimodal-instruct** está selecionada.

1. No painel principal da sessão de chat, abaixo da caixa de entrada de chat, use o botão anexar (**&#128206;**) para fazer upload do arquivo de imagem *mango.jpeg* e, em seguida, adicione o texto `What desserts could I make with this fruit?` e envie o prompt.

    ![Captura de tela da página do playground de chat.](../media/chat-playground-image.png)

1. Revise a resposta, que dará orientações relevantes de sobremesas que você pode fazer usando uma manga.

## Criar um aplicativo cliente

Agora que você implantou o modelo, pode usar a implantação em um aplicativo cliente.

### Preparar a configuração de aplicativo

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.

1. Na área **Pontos de extremidade e chaves**, verifique se a biblioteca da **Fábrica de IA do Azure** está selecionada e observe o **ponto de extremidade do projeto da Fábrica de IA do Azure**. Você usará essa cadeia de conexão para se conectar ao seu projeto em um aplicativo cliente.

1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

    Feche todas as notificações de boas-vindas para ver a home page do portal do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Você pode redimensionar ou maximizar esse painel para facilitar o trabalho.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

    > **Observação**: Se o portal solicitar que você selecione um armazenamento para persistir seus arquivos, escolha **Nenhuma conta de armazenamento necessária**, selecione a assinatura que você está usando e pressione **Aplicar**.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do projeto da Fábrica (copiada da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure) e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do modelo Phi-4-multimodal-instruct.

1. Após substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **Clique com o botão direito > Salvar** para salvar as alterações e, em seguida, use o comando **CTRL+Q** ou **Clique com o botão direito > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Escrever código para se conectar ao projeto e obter um cliente de chat para o modelo

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code chat-app.py
    ```

1. No arquivo de código, observe as instruções existentes que foram adicionadas na parte superior do arquivo para importar os namespaces do SDK necessários. Em seguida, localize o comentário **Add references**, adicione o seguinte código para referenciar os namespaces nas bibliotecas que você instalou anteriormente:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. Na função **main**, no comentário **Get configuration settings**, observe que o código carrega a cadeia de conexão do projeto e os valores do nome de implantação do modelo que você definiu no arquivo de configuração.
1. Na função **main**, no comentário **Get configuration settings**, observe que o código carrega a cadeia de conexão do projeto e os valores do nome de implantação do modelo que você definiu no arquivo de configuração.
1. Localize o comentário **Inicializar o cliente do projeto** e adicione o seguinte código para se conectar à Fábrica de IA do Azure:

    > **Dica**: Tenha cuidado para manter o nível de recuo correto no seu código.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. No comentário **Get a chat client**, adicione o seguinte código para criar um objeto cliente para conversar com um modelo:

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Escrever código para enviar um prompt de imagem baseado em URL

1. Observe que o código inclui um loop para permitir que o usuário insira um prompt até digitar "quit". Na seção de loop, localize o comentário **Get a response to image input** e adicione o seguinte código para enviar um prompt que inclua a seguinte imagem:

    ![Uma foto de uma manga.](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código; não feche-o ainda.

## Execute o aplicativo e entre no Azure.

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para entrar no Azure.

    ```
   az login
    ```

    **<font color="red">Você deve entrar no Azure, mesmo que a sessão do Cloud Shell já esteja autenticada.</font>**

    > **Observação**: na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant* . Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.
    
1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o hub da Fábrica de IA do Azure, se solicitado.

1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python chat-app.py
    ```

1. Quando solicitado, insira o seguinte prompt:

    ```
   Suggest some recipes that include this fruit
    ```

1. Analise a resposta. Em seguida, insira `quit` para sair do programa.

### Modificar o código para carregar um arquivo de imagem local

1. No editor de código do código do aplicativo, na seção de loop, localize o código adicionado anteriormente no comentário **Get a response to image input**. Em seguida, modifique o código da seguinte maneira para carregar esse arquivo de imagem local:

    ![Uma foto de uma pitaia.](../media/mystery-fruit.jpeg)

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
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código. Você também pode fechar o editor de código (**CTRL+Q**), se desejar.

1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para executar o aplicativo:

    ```
   python chat-app.py
    ```

1. Quando solicitado, insira o seguinte prompt:

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. Analise a resposta. Em seguida, insira `quit` para sair do programa.

    > **Observação**: neste aplicativo simples, não implementamos lógica para reter o histórico de conversas; portanto, o modelo tratará cada prompt como uma nova solicitação sem contexto do prompt anterior.

## Limpar

Se tiver terminado de explorar o portal do Azure AI Foundry, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Abra o [portal do Azure](https://portal.azure.com) e exiba o conteúdo do grupo de recursos em que você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
