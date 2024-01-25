---
lab:
  title: Classificar imagens usando um modelo personalizado da Visão de IA do Azure
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classificar imagens usando um modelo personalizado da Visão de IA do Azure

A Visão de IA do Azure permite treinar modelos personalizados para classificar e detectar objetos com rótulos específicos. Neste laboratório, vamos criar um modelo personalizado para classificação de imagens de frutas.

## Clone o repositório para este curso

Se você ainda não clonou o repositório de código do **Visão de IA do Azure** para o ambiente em que está trabalhando neste laboratório, siga estas etapas para fazer isso. Caso contrário, abra a pasta clonada no Visual Studio Code.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute um comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-vision` para uma pasta local (não importa qual pasta).
3. Após o repositório ter sido clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto arquivos adicionais são instalados para oferecer suporte a projetos com código C# no repositório.

    > **Observação**: caso seja solicitado que você adicione os ativos necessários para compilar e depurar, selecione **Agora não**. Se a mensagem *Projeto do Azure Functions detectado na pasta* for exibida, você pode fechar essa mensagem com segurança.

## Provisionar recursos do Azure

Caso ainda não tenha um na sua assinatura, será necessário provisionar um recurso dos **Serviços de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.
2. Na barra de pesquisa superior, pesquise *Serviços de IA do Azure*, selecione **Serviços de IA do Azure** e crie um recurso de conta multisserviço dos Serviços de IA do Azure com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos (se estiver usando uma assinatura restrita, talvez você não tenha permissão para criar um grupo de recursos, então use o que foi fornecido)*
    - **Região**: *Escolha entre Leste dos EUA, Europa Ocidental, Oeste dos EUA, Oeste dos EUA 2\**
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: Standard S0

    \*No momento, as marcas de modelo personalizado da Visão de IA do Azure 4.0 estão disponíveis apenas nessas regiões.

3. Marque as caixas de seleção necessárias e crie o recurso.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Também precisamos de uma conta de armazenamento para as imagens de treinamento.

1. Volte ao portal do Azure, procure e selecione **Contas de armazenamento** e crie uma nova conta de armazenamento com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha o mesmo grupo no qual você criou o recurso do Serviço de IA do Azure*
    - **Nome da conta de armazenamento**: customclassifySUFFIX 
        - *Observação: substitua o token `SUFFIX` por suas iniciais ou outro valor para garantir que o nome do recurso seja globalmente exclusivo.*
    - **Região**: *escolha a mesma região que usou para seu recurso do Serviço de IA do Azure*
    - **Desempenho**: padrão
    - **Redundância**: LRS (armazenamento com redundância local)
1. Enquanto sua conta de armazenamento está sendo criada, vá para o Visual Studio Code e expanda a pasta **02-image-classification**.
1. Nela, selecione **replace.ps1** e analise o código. Você verá que ele substitui o nome da sua conta de armazenamento pelo espaço reservado em um arquivo JSON (o arquivo COCO) que usamos em uma etapa posterior. Substitua o espaço reservado na primeira linha do arquivo pelo nome da sua conta de armazenamento. Salve o arquivo.
1. Clique com o botão direito do mouse e abra um Terminal Integrado em **02-image-classification** e execute o seguinte comando.

    ```powershell
    ./replace.ps1
    ```

1. Você pode analisar o arquivo COCO para garantir que o nome da sua conta de armazenamento esteja lá. Selecione **training-images/training_labels.json** e veja as primeiras entradas. No campo *absolute_url*, você deve ver algo semelhante a `"https://myStorage.blob.core.windows.net/fruit/...`.
1. Feche o arquivo JSON e o arquivo do PowerShell e volte para a janela do navegador.
1. Sua conta de armazenamento deve estar completa. Vá até sua conta de armazenamento.
1. Habilite o acesso público à conta de armazenamento. No painel à esquerda, acesse **Configuração** no grupo **Configurações** e habilite *Permitir acesso anônimo ao blob*. Selecione **Salvar**
1. No painel à esquerda, selecione **Contêineres**, crie um novo contêiner chamado `fruit` e defina o **Nível de acesso anônimo** como *Contêiner (acesso de leitura anônimo para contêineres e blobs)*.

    > **Observação**: se o **Nível de acesso anônimo** estiver desabilitado, atualize a página do navegador.

1. Navegue até `fruit` e carregue as imagens (e o arquivo JSON) em **02-image-classification\training-images** nesse contêiner.

## Criar um projeto de treinamento de modelo personalizado

Em seguida, você criará um novo projeto de treinamento para classificação de imagem personalizada no Vision Studio.

1. No navegador da Web, navegue `https://portal.vision.cognitive.azure.com/` e entre com a conta Microsoft em que você criou seu recurso de IA do Azure.
1. Selecione o bloco **Personalizar modelos com imagens** (pode ser encontrado na guia **Análise de imagem** se ele não estiver sendo exibido em seu modo de exibição padrão) e, se solicitado, selecione o recurso de IA do Azure que você criou.
1. No projeto, selecione **Adicionar novo conjunto de dados** na parte superior. Defina o  com as seguintes configurações:
    - **Nome do conjunto de dados**: training_images
    - **Tipo de modelo**: classificação de imagem
    - **Selecione o contêiner de Armazenamento de Blobs do Azure**: Selecione **Selecionar contêiner**
        - **Assinatura**: *sua assinatura do Azure*
        - **Conta de Armazenamento**: *a conta de armazenamento que você criou*
        - **Contêiner de Blobs**: fruit
    - Marque a caixa para "Permitir que o Vision Studio leia e grave no seu de armazenamento de blobs"
1. Selecione o conjunto de dados **training_images**.

Neste ponto da criação do projeto, você geralmente selecionaria **Criar Projeto de Rotulagem de Dados do Azure ML** e rotularia suas imagens, o que gera um arquivo COCO. Incentivamos tentar isso se tiver tempo, mas para os propósitos deste laboratório, já rotulamos as imagens para você e fornecemos o arquivo COCO resultante.

1. Selecione **Adicionar arquivo COCO**
1. Na lista suspensa, selecione **Importar arquivo COCO de um contêiner de Blobs**
1. Como você já conectou seu contêiner chamado `fruit`, o Vision Studio procurará um arquivo COCO. Selecione **training_labels.json** na lista suspensa e adicione o arquivo COCO.
1. Navegue até **Modelos personalizados** à esquerda e selecione **Treinar um novo modelo**. Use as configurações a seguir:
    - **Nome do modelo**: classifyfruit
    - **Tipo de modelo**: classificação de imagem
    - **Escolha o conjunto de dados de treinamento**: training_images
    - Deixe o restante como padrão e selecione **Treinar modelo**.

O treinamento pode levar algum tempo; o orçamento padrão é de até uma hora, no entanto, para esse pequeno conjunto de dados, geralmente é muito mais rápido do que isso. Selecione o botão **Atualizar** a cada dois minutos até que o status do trabalho seja *Êxito*. Selecione o modelo.

Aqui você pode ver o desempenho do trabalho de treinamento. Revise a precisão e acurácia do modelo treinado.

## Teste seu modelo personalizado

Seu modelo foi treinado e está pronto para ser testado.

1. Na parte superior da página do seu modelo personalizado, selecione **Experimentar**.
1. Selecione o modelo **classifyfruit** na lista suspensa especificando o modelo que deseja usar e navegue até a pasta **02-image-classification\test-images**.
1. Selecione cada imagem e visualize os resultados. Selecione a guia **JSON** na caixa de resultados para examinar a resposta JSON completa.

<!-- Option coding example to run-->
## Limpar os recursos

Se não estiver usando os recursos do Azure criados neste laboratório para outros módulos de treinamento, você pode excluí-los para evitar incorrer em cobranças adicionais.

1. Abra o portal do Azure em `https://portal.azure.com` e, na barra de pesquisa superior, procure os recursos criados neste laboratório.

2. Na página do recurso, selecione **Excluir** e siga as instruções para excluir o recurso. Como alternativa, você pode excluir todo o grupo de recursos para limpar todos os recursos ao mesmo tempo.