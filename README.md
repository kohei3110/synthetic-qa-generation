# 実世界データから合成QnAを生成する

## 概要
LLM/SLMのファインチューニング、RAG、または評価のために、実世界の生データからQ&A形式のデータを生成する必要があることがよくあります。しかし、既製のデータセットからではなく、ゼロからデータセットを作成する必要がある場合、多くの課題に直面します。

このハンズオンラボは、複雑な非構造化データからQnAデータセットを作成/拡張する方法を示すことで、その負担を軽減することを目的としています。サンプルは、開発者やデータサイエンティスト、そしてフィールドの人々が少しの助けを借りて試してみることができるように、ステップバイステップで進めることを目指しています。

## シナリオ

### 概要
モデルのパフォーマンスを向上させるために、ファインチューニング/RAG（Retrieval Augmented Generation）を行い、高品質なデータセットを提供します。しかし、既存のデータセットは提供されておらず、PDF、CSV、TXTなどの形式の未処理の生データしかありません。この生データには、画像、表、テキストが混在しています。

#### ステージ1. シードデータセットの構築
このタスクは、異種データをファインチューニングやRAGに適した構造化形式に前処理して変換することです。これには、さまざまなファイル形式からテキストを抽出してクリーニングし、必要に応じてAzure AIサービスを使用して表や画像をテキストに変換することが含まれます。このデータセットは、ファインチューニングやRAGのためのシードデータセットとして使用され、ドメイン固有のユースケースのパフォーマンスを向上させるためのベースラインとして使用されます。

#### ステージ2. データ拡張（オプション）
生成されたデータセットでファインチューニングを行った後、ベースラインが確立されましたが、データが不足しているため（例：データセットにサンプルが1,000個しかない）、パフォーマンスの向上が必要です。この場合、データ拡張技術を適用して合成データセットを作成し、パフォーマンスを向上させる必要があります。データ拡張技術は、Microsoftが発表した代表的な技術であるEvol-Instruct、GLAN（Generalized Instruction Tuning）、およびAuto Evol-Instructを利用します。

### 実際のアプリケーションの例
以下は、RAGなしでGPT-4oのファインチューニング前後の結果をお客様のPoCで比較したものです。GPT-4oは2024年7月現在、少数のお客様にプライベートプレビューとして提供されています。これは、PoCのために16の質問と回答のセットを作成し、Azure AIスタジオで**類似性、一貫性、流暢さ**の3つの指標を比較した結果です。指標の値は1-5のスケールで、高い値が良いことを示します。

![evaluation-sample](./imgs/evaluation-sample.png)

## 要件
開始する前に、以下の要件を満たしていることを確認してください：

- Azure OpenAIサービスへのアクセス - [こちら](https://go.microsoft.com/fwlink/?linkid=2222006)からアクセスを申請できます
- Azure AI Studioプロジェクト - [aka.ms/azureaistudio](https://aka.ms/azureaistudio)にアクセスしてプロジェクトを作成します
- Azure AI Document Intelligence（v4.0 - 2024-02-29プレビュー） - 詳細は[こちら](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-4.0.0)をご覧ください

アカウントに合わせて`.env`ファイルを変更することを忘れないでください。`.env.sample`を`.env`にリネームするか、コピーして使用してください。

## コンテンツ

### ステージ1. シードデータセットの構築
![diagram1](./imgs/diagram1.png)

Azure OpenAI GPT-4oを使用して、与えられた生データをモデルトレーニング/RAG/評価に使用できるデータに変換します。`make_qa_multimodal_pdf_docai.ipynb`が最も推奨されます。ただし、このコードのロジックが複雑だと感じる場合や、ファイルの内容が画像やテキストのみで構成されている場合は、他のJupyterノートブックを先に試してみてください。
**[seed](seed)**フォルダー内のJupyterノートブックを実行します。

#### PDF
- `make_qa_multimodal_pdf_docai.ipynb`: （推奨）Azure AI Document Intelligenceを使用して複雑なPDFからQnA合成データセットを生成します。
- `make_qa_multimodal_pdf_oss.ipynb`: オープンソース（このハンズオンのためのUnstructuredツールキット）を使用して複雑なPDFからQnA合成データセットを生成します。このファイルを実行するには、まず`startup_unstructured.sh`で必要なパッケージをインストールする必要があります。インストールには数分かかります。
- `make_qa_only_image_multiple_pdf.ipynb`: 複数のPDFからQnA合成データセットを生成します - 画像が多いPDF。
- `make_qa_only_image_pdf.ipynb`: 画像が多いPDFからQnA合成データセットを生成します。

#### CSV
- `make_qa_csv.ipynb`: これは一般的なケースです。CSVLoaderを使用して読み込みとチャンク化を行うことで、QnAデータセットを作成するのは難しくありません。
- `make_qa_image_url_csv.ipynb`: これはもう一つの一般的なケースです。画像URL情報が含まれている場合、そのURLをその画像の要約結果に変更します。

### ステージ2. データ拡張（オプション）
Microsoftの研究を活用して、より高品質で複雑なデータを生成します。ステージ1でベースラインを確立したら、このステップを試してさらに良い結果を得ることができます。Evolve-InstructとGLANの概念を利用することで、特定の業界/技術分野に特化したLLMにファインチューニングすることができます。

#### [Evolve-Instruct](evolve-instruct/README.md)

![diagram2](./imgs/diagram2.png)

ステージ1で作成したシードデータセットに基づいてデータ拡張を行うことができます。詳細は**[evolve-instruct/README](evolve-instruct/README.md)**をご覧ください。

#### [GLAN（Generalized Instruction Tuning）](glan-instruct/README.md)

![diagram3](./imgs/diagram3.png)

GLANはステージ1を経ることなく独立して実行できます。これはすべての一般化されたドメインをカバーしているためです。詳細は**[glan-instruct/README](glan-instruct/README.md)**をご覧ください。

## 始め方
どのオプションでも構いませんが、以下の指示を参照してください：
- PoC/MVPでこのハンズオンを使用したいエンジニアや実務者には、オプション1をお勧めします。
- ワークショップでこのハンズオンを使用したいインストラクターには、オプション2をお勧めします。
- プロダクションを立ち上げたいフィールドの開発者には、オプション3をお勧めします。

### オプション1. Azure AI StudioまたはAzure ML Studio
コンピュートインスタンスを作成します。コード開発には`Standard_DS11_v2`（2コア、14GB RAM、28GBストレージ、GPUなし）をお勧めします。

複雑なPDFを処理するためにUnstructuredツールキットを使用したい場合は、インスタンスのスタートアップスクリプトに`startup_unstructured.sh`を必ず含めてください。

### オプション2. GitHub Codespace
Codespaceプロジェクトに接続して新しいプロジェクトを開始します。ハンズオンに必要な環境はdevcontainerを通じて自動的に構成されるため、Jupyterノートブックを実行するだけです。

### オプション3. ローカルPC
ローカルPCに必要なパッケージを`pip install -r requirements.txt`でインストールして開始します。

## 参考文献
- Evolve-Instruct: https://arxiv.org/pdf/2304.12244
- GLAN（Generalized Instruction Tuning）: https://arxiv.org/pdf/2402.13064
- Auto Evolve-Instruct: https://arxiv.org/pdf/2406.00770

## 貢献

このプロジェクトは貢献と提案を歓迎します。ほとんどの貢献には、貢献者ライセンス契約（CLA）に同意する必要があります。これにより、あなたが貢献する権利を持ち、実際に貢献を使用する権利を私たちに与えることを宣言します。詳細はhttps://cla.opensource.microsoft.comをご覧ください。

プルリクエストを送信すると、CLAボットが自動的にCLAを提供する必要があるかどうかを判断し、適切にPRを装飾します（例：ステータスチェック、コメント）。ボットの指示に従うだけです。これは、すべてのリポジトリで一度だけ行う必要があります。

このプロジェクトは[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/)を採用しています。詳細については[Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/)をご覧いただくか、[opencode@microsoft.com](mailto:opencode@microsoft.com)にお問い合わせください。

## 商標

このプロジェクトには、プロジェクト、製品、またはサービスの商標やロゴが含まれている場合があります。Microsoftの商標やロゴの許可された使用は、[Microsoftの商標およびブランドガイドライン](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general)に従う必要があります。このプロジェクトの変更バージョンでMicrosoftの商標やロゴを使用する場合、混乱を引き起こしたり、Microsoftのスポンサーシップを暗示したりしてはなりません。第三者の商標やロゴの使用は、それらの第三者のポリシーに従う必要があります。

## ライセンス概要
このサンプルコードはMIT-0ライセンスの下で提供されています。詳細はLICENSEファイルをご覧ください。