---
slug: /lfx-mentorship-enriching-awschaos
title: From a Newbie in Software Engineering to a Graduated LFX-Mentee
authors: debabratapanigrahi
image: /img/blog/mentorship_experience.png
tags: [Chaos Mesh, Chaos Engineering, LFX Mentorship, AWS Chaos]
---

![LFXメンターシップ体験](/img/blog/mentorship_blog.jpeg)

[私は](https://mentorship.lfx.linuxfoundation.org/mentee/6a0bf7de-9e18-4acb-9a66-f5fecdbeb42e)、インドの[国立工科大学ルールケラ校](https://nitrkl.ac.in/)のバイオテクノロジー・医療工学科に在籍する生物医学工学専攻の学部生です。プログラミングに興味を持ったきっかけは純粋な好奇心からで、独学の道のりにはさまざまな困難がありました。しかし、オープンソースへの貢献を始めてからは、初心者にも優しい環境に恵まれ、技術スタックを深く理解する手助けをしてくれる多くの人々に出会いました。

<!--truncate-->

![img1](/img/blog/mentroship_blog1.png)

## 応募までの道のり

2021年春、LFXメンターシッププログラムの存在を知り、すべての[プロジェクト](https://github.com/cncf/mentoring/blob/master/lfx-mentorship/2021/01-Spring/README.md)を閲覧しましたが、ほとんどの用語に馴染みがなく、初心者向けではないと感じて困惑しました。その後、プログラムの[ドキュメント](https://docs.linuxfoundation.org/lfx/mentorship)や[FAQ](https://docs.linuxfoundation.org/lfx/mentorship/mentorship-faqs)を読み、記載されている手順に従って、Docker、AWS、Pythonなど、私が慣れ親しんだ技術スタックを使用する興味深いプロジェクトにいくつか応募しました。

その後、[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)が提供する両プロジェクトに応募し、CVとカバーレターを即座の課題として提出しました。数日後、メンターから追加課題の提出に関するメールを受け取りました。

![img2](/img/blog/mentorship_blog2.png)

上記の課題を完了し、ファイルをGitHubにアップロードして、メンターとリンクを共有しました。

## 選考とメンティーとしての初期の日々

:::note

2022-10-24: https://www.oreilly.com/online-learning/leveraging-katacoda-technology.html および [#356](https://github.com/chaos-mesh/website/pull/356) により、インタラクティブチュートリアルは一時的に利用できません。

:::

メンターシッププログラムに選ばれたことを知らせるメールを受け取った日のことを鮮明に覚えています。初めてのオープンソースプログラムへの参加だったため、非常に喜びました。CNCFからも選考結果のメールを受け取り、プログラムのメンティーとして受け入れられたことを嬉しく思いました。

![img3](/img/blog/mentorship_blog4.png)

メンターとともに、Slackを通じてのコミュニケーション方法を決定しました。また、KubernetesとGOlangに関する知識について尋ねられ、どちらにもあまり詳しくなかったため、メンターはいくつかのリソースを提案し、2週間の期間を設けてそれらを学ぶように指示しました。その間、これらの技術に慣れるためのいくつかの実験も計画されました。

Kubernetesに慣れてくると、Chaos Meshの探索を始め、`インタラクティブチュートリアル`を完了しました。これにより、Chaos Meshの使用方法についてより明確な理解を得ることができました。その後、[hello-world chaos](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/development_guides/develop_a_new_chaos)を実装し、コントローラーとCRDについてさらに学びました。これらはChaos Meshの最も重要な部分とされています。また、ボイラープレートコードや[kube-builderクライアント](https://github.com/kubernetes-sigs/kubebuilder)について知り、それらをスキャフォールディングに使用する方法や、独自のコントローラーを書く方法について学びました。

初期の実験期間を経てプロジェクトについてより理解を深めた後、Chaos Meshへのアップストリーム貢献に慣れるため、いくつかの「good first issue」に取り組み始めました。

![img4](/img/blog/mentorship_blog3.png)

私の貢献の一つとして、stress-chaosにマルチコンテナサポートを追加しようと試みました。これは以前は不可能でした。実装には成功したものの、他の機能に影響を与えてしまい、次のリリースにマージすることはできませんでした。さらに、2.0.0リリースではこのリファクタリングは既に行われていたため、この貢献は私とメンターの両方にとって学びの経験となりました。その後、私たちはより慎重になり、新機能を実装する際にはまず[RFC](https://github.com/chaos-mesh/rfcs)を提出し、他のコントリビューターと議論してから着手するようになりました。

## AWS Chaosへの貢献

当初、このプロジェクトの一環として1種類のAWS Chaosを実装するよう依頼されましたが、調査を進めるうちに[awsssmchaosrunner](https://github.com/amzn/awsssmchaosrunner)を見つけ、その機能を考慮してChaos Meshに統合したいと考えました。

私たちはこれを2つの部分に分けて計画しました。1つはawsssmchaosrunnerと統合する「[runner thing](https://github.com/STRRL/awsssmchaosrunner-cli)」プロジェクトで、Kotlinで記述し、Dockerイメージをビルドする部分です。

もう1つはAWS Chaosの定義とその[コントローラー](https://github.com/chaos-mesh/chaos-mesh/pull/1919)で、Goで記述されます。AWS Chaosのコントローラーは「Kotlin CLIイメージ」を含むPodを作成し、AWSにコマンドを送信します。

## その他の機会

メンターシップ終盤に、Chaos Meshの[コミュニティミーティング](https://www.youtube.com/watch?v=ElG0pHRoXwI&t=2s)に招待され、自分のプロジェクトを発表しました。

その後、2021年6月25-26日に仮想開催された[Kubernetes Community Days Bangalore](https://community.cncf.io/events/details/cncf-kcd-bengaluru-presents-kubernetes-community-days-bengaluru/)のCFPに応募し、スピーカーとして選出され、現在はそこで講演する準備を整えています。

## 卒業と次のステップ

やったね！！12週間後、メンターの[Zhou Zhiqiang](https://mentorship.lfx.linuxfoundation.org/mentor/e78b3177-160c-4566-9f3d-8fc9b2ec3cea)氏の指導のおかげで、無事にプログラムを卒業することができました。彼がいなければ、この成果は得られなかったでしょう。

Chaos Meshコミュニティでは素晴らしい時間を過ごし、旅路を通じてサポートしてくれた素晴らしいメンバーたちに感謝しています。今後もこのプロジェクトにさらに貢献し、コミュニティでより積極的に関わっていきたいと考えています。

## Chaos Meshコミュニティに参加しよう

Chaos Meshに参加して詳しく知りたい方は、[CNCF Slackワークスペース](https://slack.cncf.io/)の#project-chaos-meshチャンネルか、[GitHub](https://github.com/chaos-mesh/chaos-mesh)をご覧ください。