---
lab:
  title: Classificar imagens usando um modelo personalizado da Visão de IA do Azure
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classificar imagens usando um modelo personalizado da Visão de IA do Azure

A Visão de IA do Azure permite treinar modelos personalizados para classificar e detectar objetos com rótulos específicos. Neste laboratório, vamos criar um modelo personalizado para classificação de imagens de frutas.

## Clone o repositório para este curso

Se você ainda não clonou o repositório de códigos da **Visão de IA do Azure** para o ambiente em que está trabalhando neste laboratório, siga estas etapas. Caso já tenha feito isso, abra a pasta clonada no Visual Studio Code.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute um comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` para uma pasta local (não importa qual).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto outros arquivos adicionais são instalados para que haja suporte a projetos com código C# no repositório.

    > **Observação**: caso seja solicitado que você adicione os ativos necessários para compilar e depurar, selecione **Agora não**. Caso receba a mensagem informando que *foi detectado um projeto de função do Azure na pasta*, ela pode ser fechada com segurança.

## Provisionar recursos do Azure

Caso ainda não tenha um na sua assinatura, será necessário provisionar um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, busque *serviços de IA do Azure*, selecione **Serviços de IA do Azure** e crie um recurso de conta multisserviço correspondente com as seguintes configurações:
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
    - **Grupo de recursos**: *escolha o mesmo grupo em que você criou o recurso do Serviço de IA do Azure*
    - **Nome da conta de armazenamento**: customclassifySUFFIX 
        - *Observação: substitua o token `SUFFIX` pelas suas iniciais ou algum outro valor para garantir que o nome do recurso seja globalmente exclusivo.*
    - **Região**: *escolha a mesma região que usou para seu recurso do Serviço de IA do Azure*
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
1. Habilite o acesso público à conta de armazenamento. No painel à esquerda, acesse **Configuração** no grupo **Configuraçoes** e habilite *Permitir acesso anônimo ao blob*. Selecione **Salvar**
1. No painel à esquerda, selecione **Contêineres**, crie um novo contêiner chamado `fruit` e defina o **Nível de acesso anônimo** como *Contêiner (acesso de leitura anônimo para contêineres e blobs)*.

    > **Observação**: se o **nível de acesso anônimo** estiver desabilitado, atualize a página do navegador.

1. Navegue até `fruit` e carregue as imagens (e o único arquivo JSON) em **Labfiles/02-image-classification/training-images** nesse contêiner.

## Criar um projeto de treinamento de modelo personalizado

Em seguida, você criará um novo projeto de treinamento para classificação de imagens no Vision Studio de forma personalizada.

1. No navegador da Web, acesse `https://portal.vision.cognitive.azure.com/` e entre com a conta da Microsoft em que você criou seu recurso de IA do Azure.
1. Selecione o bloco **Personalizar modelos com imagens** (que fica na guia **Análise de imagem** caso não seja mostrado no modo de exibição padrão) e, se solicitado, selecione o recurso de IA do Azure que você criou.
1. No projeto, selecione **Adicionar novo conjunto de dados** na parte superior. Defina o  com as seguintes configurações:
    - **Nome do conjunto de dados**: training_images
    - **Tipo de modelo**: classificação de imagens
    - **Selecione o contêiner de armazenamento de blobs do Azure**: escolha **Selecionar contêiner**
        - **Assinatura**: *sua assinatura do Azure*
        - **Conta de armazenamento**: *a conta de armazenamento que você criou*
        - **Contêiner de blobs**: fruta
    - Selecione a caixa para permitir que o Vision Studio leia e grave no contêiner de armazenamento de blobs
1. Selecione o conjunto de dados **training_images**.

Neste ponto da criação do projeto, você geralmente selecionaria **Criar um projeto de rotulagem de dados do Azure ML** e rotularia suas imagens para gerar um arquivo COCO. Tente fazer isso se tiver tempo, mas para este laboratório, nós já rotulamos as imagens para você e fornecemos o arquivo COCO resultante.

1. Selecione **Adicionar arquivo COCO**
1. Na lista suspensa, selecione **Importar um arquivo COCO de um contêiner de blobs**
1. Como você já conectou seu contêiner chamado `fruit`, o Vision Studio vai procurar um arquivo COCO nele. Selecione **training_labels.json** na lista suspensa e adicione o arquivo COCO.
1. Navegue até **Modelos personalizados** à esquerda e selecione **Treinar um novo modelo**. Use as configurações a seguir:
    - **Nome do modelo**: classifyfruit
    - **Tipo de modelo**: classificação de imagens
    - **Escolha o conjunto de dados de treinamento**: training_images
    - Deixe o restante como padrão e selecione **Treinar modelo**

O treinamento pode levar algum tempo. O orçamento padrão é de até uma hora, mas costuma ser mais rápido porque esse conjunto de dados é pequeno. Selecione o botão **Atualizar** a cada dois minutos até que o status do trabalho seja *Bem-sucedido*. Selecione o modelo.

Aqui você pode ver o desempenho do trabalho de treinamento. Revise a precisão e exatidão do modelo treinado.

## Testar seu modelo personalizado

Seu modelo foi treinado e está pronto para ser testado.

1. Na parte superior da página do seu modelo personalizado, selecione **Testar**.
1. Selecione o modelo **classifyfruit** na lista suspensa e navegue até a pasta **02-image-classification\test-images**.
1. Selecione cada imagem e veja os resultados. Selecione a guia **JSON** na caixa de resultados para examinar a resposta JSON completa.

<!-- Option coding example to run-->
## Limpar os recursos

Se você não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, pode excluí-los para evitar cobranças adicionais.

1. Abra o portal do Azure em `https://portal.azure.com` e, na barra de pesquisa superior, procure os recursos criados neste laboratório.

2. Na página do recurso, selecione **Excluir** e siga as instruções. Outra opção é excluir o grupo de recursos inteiro para limpar tudo ao mesmo tempo.