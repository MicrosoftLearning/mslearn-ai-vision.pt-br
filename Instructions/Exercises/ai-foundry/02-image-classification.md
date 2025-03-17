---
lab:
  title: Classificar imagens com um modelo personalizado da Visão de IA do Azure
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classificar imagens com um modelo personalizado da Visão de IA do Azure

A Visão de IA do Azure permite treinar modelos personalizados para classificar e detectar objetos com rótulos especificados. Neste laboratório, criaremos um modelo personalizad de classificação de imagem para classificar imagens de frutas.

## Clone o repositório para este curso

Se você ainda não clonou o repositório de código de **Visão de IA do Azure** para o ambiente em que está trabalhando neste laboratório, siga estas etapas para fazer isso. Caso contrário, abra a pasta clonada no Visual Studio Code.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar ativos necessários para compilar e depurar, selecione **Agora não**. Se você for surpreendido com a mensagem *Detectado um projeto de função do Azure na pasta*, você pode fechar essa mensagem com segurança.

## Provisionar recursos do Azure

Caso ainda não tenha um na sua assinatura, provisione um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, pesquise *serviços de IA do Azure*, selecione **Serviços de IA do Azure** e crie um recurso de conta multisserviço dos serviços de IA do Azure com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se estiver usando uma assinatura restrita, talvez você não tenha permissão para criar um grupo de recursos, então use o que foi fornecido)*
    - **Região**: *escolha entre Leste dos EUA, Oeste da Europa e Oeste dos EUA 2\**
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0

    \*No momento, as marcas de modelo personalizado da Visão de IA do Azure 4.0 estão disponíveis apenas nessas regiões.

3. Marque as caixas de seleção necessárias e crie o recurso.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Também precisamos de uma conta de armazenamento para as imagens de treinamento.

1. No portal do Azure, pesquise e selecione **Contas de armazenamento** e crie uma nova conta desse tipo com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha o grupo de recursos no qual você criou o recurso de serviço de IA do Azure*.
    - **Nome da conta de armazenamento**: customclassifySUFFIX 
        - *Observação: substitua o token `SUFFIX` por suas iniciais ou outro valor para garantir que o nome do recurso seja globalmente exclusivo.*
    - **Região**: *escolha a mesma região que você usou para seu recurso do serviço de IA do Azure*
    - **Serviço primário**: Armazenamento de Blobs do Azure ou Azure Data Lake Storage Gen2
    - **Carga de trabalho primária**: outro
    - **Desempenho**: padrão
    - **Redundância**: LRS (armazenamento com redundância local)
1. Enquanto sua conta de armazenamento estiver sendo criada, abra o Visual Studio Code e acesse a pasta **Labfiles/02-image-classification**.
1. Nela, selecione **replace.ps1** e revise o código. Você verá que o nome da sua conta de armazenamento foi substituído pelo espaço reservado em um arquivo JSON (o arquivo COCO) que vamos usar em uma etapa posterior. Substitua o espaço reservado do arquivo pelo nome da sua conta de armazenamento *apenas na primeira linha*. Salve o arquivo.
1. Clique com o botão direito do mouse na pasta **02-image-classification** e abra um terminal Integrado. Execute o comando a seguir.

    ```powershell
    ./replace.ps1
    ```

1. Você pode revisar o arquivo COCO para confirmar se o nome da sua conta de armazenamento está lá. Selecione **training-images/training_labels.json** e veja as primeiras entradas. No campo *absolute_url*, você deve notar algo parecido com *"https://myStorage.blob.core.windows.net/fruit/...*. Se não encontrar a alteração esperada, atualize apenas o primeiro espaço reservado no script do PowerShell.
1. Feche o arquivo JSON e o arquivo do PowerShell e volte para a janela do navegador.
1. Sua conta de armazenamento deve estar completa. Vá até sua conta de armazenamento.
1. Habilite o acesso público à conta de armazenamento. No painel à esquerda, acesse **Configuração** no grupo **Configurações** e habilite *Permitir acesso anônimo ao blob*. Selecione **Salvar**
1. No painel à esquerda, em **Armazenamento de dados**, selecione **Contêineres**, crie um novo contêiner chamado `fruit` e defina o **Nível de acesso anônimo** como *Contêiner (acesso de leitura anônimo para contêineres e blobs)*.

    > **Observação**: se o **nível de acesso anônimo** estiver desabilitado, atualize a página do navegador.

1. Navegue até `fruit`, selecione **Upload** e faça upload das imagens (e o único arquivo JSON) em **Labfiles/02-image-classification/training-images** nesse contêiner.

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
