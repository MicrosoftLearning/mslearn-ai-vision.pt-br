---
lab:
  title: Classificar imagens com um modelo personalizado da Visão de IA do Azure
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classificar imagens com um modelo personalizado da Visão de IA do Azure

A Visão de IA do Azure permite treinar modelos personalizados para classificar e detectar objetos com rótulos especificados. Neste laboratório, criaremos um modelo personalizad de classificação de imagem para classificar imagens de frutas.

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
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Também precisamos de uma conta de armazenamento para as imagens de treinamento.

1. No portal do Azure, pesquise e selecione **Contas de armazenamento** e crie uma nova conta desse tipo com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de Recursos**: *Escolha o mesmo grupo de recursos no qual você criou seu recurso de Visão Personalizada*
    - **Nome da conta de armazenamento**: customclassifySUFFIX 
        - *Observação: substitua o token `SUFFIX` por suas iniciais ou outro valor para garantir que o nome do recurso seja globalmente exclusivo.*
    - **Região**: *escolha a mesma região que você usou para seu recurso do serviço de IA do Azure*
    - **Serviço primário**: Armazenamento de Blobs do Azure ou Azure Data Lake Storage Gen2
    - **Carga de trabalho primária**: outro
    - **Desempenho**: padrão
    - **Redundância**: LRS (armazenamento com redundância local)

1. Depois de implantar o recurso, selecione **Ir para o recurso**.
1. Habilite o acesso público à conta de armazenamento. No painel à esquerda, acesse **Configuração** no grupo **Configurações** e habilite *Permitir acesso anônimo ao blob*. Selecione **Salvar**
1. No painel à esquerda, em **Armazenamento de dados**, selecione **Contêineres**, crie um novo contêiner chamado `fruit` e defina o **Nível de acesso anônimo** como *Contêiner (acesso de leitura anônimo para contêineres e blobs)*.

    > **Observação**: se o **nível de acesso anônimo** estiver desabilitado, atualize a página do navegador.
   
## Clone o repositório para este curso

Os arquivos de imagem para treinamento do modelo foram fornecidos em um repositório do GitHub. Você clonará o repositório e carregará as imagens em sua conta de armazenamento usando o Cloud Shell no Portal do Azure. 

> **Dica**: Se você já clonou o repositório **mslearn-ai-vision** recentemente, pode ignorar a tarefa de clonagem. Caso contrário, siga estas etapas para clonar o repositório em seu ambiente de desenvolvimento.

1. No Portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Depois que o repositório tiver sido clonado, navegue até a pasta que contém os arquivos do exercício:  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. Execute o comando `code replace.ps1` e examine o código. Você verá que o nome da sua conta de armazenamento foi substituído pelo espaço reservado em um arquivo JSON (o arquivo COCO) que vamos usar em uma etapa posterior.
1. Substitua o espaço reservado do arquivo pelo nome da sua conta de armazenamento *apenas na primeira linha*.
1. Após substituir o espaço reservado, no editor de código, use o comando **CTRL+S** ou **clique com o botão direito > Salvar** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** ou **clique com o botão direito > Sair** para fechar o editor de código mantendo a linha de comando do Cloud Shell aberta.
1. Execute o script com o seguinte comando:

    ```powershell
    ./replace.ps1
    ```

1. Você pode revisar o arquivo COCO para confirmar se o nome da sua conta de armazenamento está lá. Execute `code training-images/training_labels.json` e exiba as primeiras entradas. No campo *absolute_url*, você deve notar algo parecido com *"https://myStorage.blob.core.windows.net/fruit/...*. Se não encontrar a alteração esperada, atualize apenas o primeiro espaço reservado no script do PowerShell.
1. Feche o editor de código.
1. Execute o seguinte comando, substituindo `<your-storage-account>` pelo nome da sua conta de armazenamento, para carregar do conteúdo da pasta **training-images** para o container `fruit` que você criou anteriormente.

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. Abra o contêiner `fruit` e verifique se os arquivos foram carregados corretamente.

## Criar um projeto de treinamento de modelo personalizado

Em seguida, você criará um novo projeto de treinamento para classificação de imagem personalizada no Vision Studio.

1. No navegador da Web, navegue até `https://portal.vision.cognitive.azure.com/` e entre com a conta Microsoft onde você criou seu recurso de IA do Azure.
1. Selecione o bloco **Personalizar modelos com imagens** (que fica na guia **Análise de imagem** caso não seja mostrado na exibição padrão).
1. Selecione a conta dos Serviços de IA do Azure que você criou.
1. No projeto, selecione **Adicionar novo conjunto de dados** na parte superior. Defina o  com as seguintes configurações:
    - **Nome do conjunto de dados**: training_images
    - **Tipo de modelo**: classificação de imagem
    - **Escolha contêiner de armazenamento de blobs do Azure**: escolha **Selecionar contêiner**
        - **Assinatura**: *sua assinatura do Azure*
        - **Conta de armazenamento**: *a conta de armazenamento que você criou*
        - **Contêiner de blob**: fruta
    - Marque a caixa para "Permitir que o Vision Studio leia e escreva no armazenamento de blobs"
1. Selecione o conjunto de dados **training_images**.

Neste ponto da criação do projeto, você geralmente selecionaria **Criar Projeto de Rotulagem de Dados do Azure ML** e rotularia suas imagens, o que gera um arquivo COCO. Incentivamos tentar isso se tiver tempo, mas para os propósitos deste laboratório, já rotulamos as imagens para você e fornecemos o arquivo COCO resultante.

1. Selecione **Adicionar arquivo COCO**
1. Na lista suspensa, selecione **Importar arquivo COCO de um contêiner de blob**
1. Como você já conectou seu contêiner `fruit`, o Vision Studio procura um arquivo COCO. Selecione **training_labels.json** na lista suspensa e adicione o arquivo COCO.
1. Navegue até **Modelos personalizados** à esquerda e selecione **Treinar um novo modelo**. Use as configurações a seguir:
    - **Nome do modelo**: classifyfruit
    - **Tipo de modelo**: classificação de imagem
    - **Escolha o conjunto de dados de treinamento**: training_images
    - Deixe o restante como padrão e selecione **Treinar um modelo**

O treinamento pode levar algum tempo – o orçamento padrão é de até uma hora, no entanto, para esse pequeno conjunto de dados, geralmente é muito mais rápido do que isso. Selecione o botão **Atualizar** a cada dois minutos até que o status do trabalho seja *Bem-sucedido*. Selecione o modelo.

Aqui você pode ver o desempenho do trabalho de treinamento. Revise a precisão e exatidão do modelo treinado.

## Testar seu modelo personalizado

Seu modelo foi treinado e está pronto para ser testado.

1. Na parte superior da página do seu modelo personalizado, selecione **Experimentar**.
1. Selecione o modelo **classifyfruit** na lista suspensa especificando o modelo que deseja usar, e navegue até a pasta **02-image-classification\test-images**.
1. Selecione cada imagem e visualize os resultados. Selecione a guia **JSON** na caixa de resultados para examinar a resposta JSON completa.

<!-- Option coding example to run-->
## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, poderá excluí-los para evitar incorrer em cobranças adicionais.

1. Abra o portal do Azure em `https://portal.azure.com`,e na barra de pesquisa superior, procure os recursos criados neste laboratório.

2. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso. Como alternativa, você pode excluir todo o grupo de recursos para limpar todos os recursos ao mesmo tempo.
