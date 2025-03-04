---
title: About GitHub-hosted runners
intro: '{% data variables.product.prodname_dotcom %}は、ワークフローを実行するためのホストされた仮想マシンを提供します。 仮想マシンには、{% data variables.product.prodname_actions %}で使用できるツール、パッケージ、および設定の環境が含まれています。'
redirect_from:
  - /articles/virtual-environments-for-github-actions
  - /github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions
  - /github/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners
  - /actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners
  - /actions/reference/virtual-environments-for-github-hosted-runners
  - /actions/reference/software-installed-on-github-hosted-runners
  - /actions/reference/specifications-for-github-hosted-runners
miniTocMaxHeadingLevel: 3
versions:
  fpt: '*'
  ghes: '*'
  ghec: '*'
shortTitle: GitHub-hosted runners
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Overview of {% data variables.product.prodname_dotcom %}-hosted runners

Runners are the machines that execute jobs in a {% data variables.product.prodname_actions %} workflow. For example, a runner can clone your repository locally, install testing software, and then run commands that evaluate your code.

{% data variables.product.prodname_dotcom %} provides runners that you can use to run your jobs, or you can [host your own runners](/actions/hosting-your-own-runners/about-self-hosted-runners). Each {% data variables.product.prodname_dotcom %}-hosted runner is a new virtual machine (VM) hosted by {% data variables.product.prodname_dotcom %} with the runner application and other tools preinstalled, and is available with Ubuntu Linux, Windows, or macOS operating systems. {% data variables.product.prodname_dotcom %}ホストランナーを使用すると、マシンのメンテナンスとアップグレードが自動的に行われます。

{% ifversion not ghes %}

## {% data variables.product.prodname_dotcom %}ホストランナーの利用

To use a {% data variables.product.prodname_dotcom %}-hosted runner, create a job and use `runs-on` to specify the type of runner that will process the job, such as `ubuntu-latest`, `windows-latest`, or `macos-latest`. For the full list of runner types, see "[Supported runners and hardware resources](/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)."

When the job begins, {% data variables.product.prodname_dotcom %} automatically provisions a new VM for that job. All steps in the job execute on the VM, allowing the steps in that job to share information using the runner's filesystem. You can run workflows directly on the VM or in a Docker container. When the job has finished, the VM is automatically decommissioned.

The following diagram demonstrates how two jobs in a workflow are executed on two different {% data variables.product.prodname_dotcom %}-hosted runners.

![Two runners processing separate jobs](/assets/images/help/images/overview-github-hosted-runner.png)

The following example workflow has two jobs, named `Run-npm-on-Ubuntu` and `Run-PSScriptAnalyzer-on-Windows`. When this workflow is triggered, {% data variables.product.prodname_dotcom %} provisions a new virtual machine for each job.

- The job named `Run-npm-on-Ubuntu` is executed on a Linux VM, because the job's `runs-on:` specifies `ubuntu-latest`.
- The job named `Run-PSScriptAnalyzer-on-Windows` is executed on a Windows VM, because the job's `runs-on:` specifies `windows-latest`.

```yaml{:copy}
name: Run commands on different operating systems
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  Run-npm-on-Ubuntu:
    name: Run npm on Ubuntu
    runs-on: ubuntu-latest
    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - uses: {% data reusables.actions.action-setup-node %}
        with:
          node-version: '14'
      - run: npm help

  Run-PSScriptAnalyzer-on-Windows:
    name: Run PSScriptAnalyzer on Windows
    runs-on: windows-latest
    steps:
      - uses: {% data reusables.actions.action-checkout %}
      - name: Install PSScriptAnalyzer module
        shell: pwsh
        run: |
          Set-PSRepository PSGallery -InstallationPolicy Trusted
          Install-Module PSScriptAnalyzer -ErrorAction Stop
      - name: Get list of rules
        shell: pwsh
        run: |
          Get-ScriptAnalyzerRule
```

While the job runs, the logs and output can be viewed in the {% data variables.product.prodname_dotcom %} UI:

![Job output in the Actions UI](/assets/images/help/repository/actions-runner-output.png)

{% data reusables.actions.runner-app-open-source %}

## サポートされているランナーとハードウェアリソース

Windows および Linux 仮想マシンのハードウェア仕様:
- 2-core CPU (x86_64)
- 7 GB of RAM
- 14 GB of SSD space

macOS 仮想マシンのハードウェア仕様:
- 3-core CPU (x86_64)
- 14 GB of RAM
- 14 GB of SSD space

{% data reusables.actions.supported-github-runners %}

ワークフローログには、ジョブの実行に使用されたランナーが一覧表示されます。 詳しい情報については、「[ワークフロー実行の履歴を表示する](/actions/managing-workflow-runs/viewing-workflow-run-history)」を参照してください。

## サポートされているソフトウェア

{% data variables.product.prodname_dotcom %} ホストランナーに含まれているソフトウェアツールは毎週更新されます。 The update process takes several days, and the list of preinstalled software on the `main` branch is updated after the whole deployment ends.
### Preinstalled software

ワークフローログには、正確なランナーにプレインストールされているツールへのリンクが含まれています。 ワークフローログでこの情報を見つけるには、[`Set up job`] セクションを展開します。 そのセクションの下で、[`Virtual Environment`] セクションを展開します。 The link following `Included Software` will describe the preinstalled tools on the runner that ran the workflow. ![Installed software link](/assets/images/actions-runner-installed-software-link.png) 詳しい情報については、「[ワークフローの実行履歴を表示する](/actions/managing-workflow-runs/viewing-workflow-run-history)」を参照してください。

For the overall list of included tools for each runner operating system, see the links below:

* [Ubuntu 22.04 LTS](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu2204-Readme.md)
* [Ubuntu 20.04 LTS](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu2004-Readme.md)
* [Ubuntu 18.04 LTS](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu1804-Readme.md)
* [Windows Server 2022](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2022-Readme.md)
* [Windows Server 2019](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2019-Readme.md)
* [macOS 12](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-12-Readme.md)
* [macOS 11](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-11-Readme.md)
* [macOS 10.15](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-10.15-Readme.md)

{% data variables.product.prodname_dotcom %}ホストランナーには、オペレーティングシステムのデフォルトの組み込みツールに加え、上のリファレンスのリスト内のパッケージにが含まれています。 たとえば、Ubuntu及びmacOSのランナーには、`grep`、`find`、`which`やその他のデフォルトのツールが含まれています。

### Using preinstalled software

アクションを使用して、ランナーにインストールされているソフトウェアと対話することをお勧めします。 このアプローチにはいくつかのメリットがあります。
- アクションでは通常、バージョンの選択、引数を渡す機能、パラメータなどの機能が提供されています
- これにより、ソフトウェアの更新に関係なく、ワークフローで使用されるツールのバージョンが同じままになります

リクエストしたいツールがある場合、[actions/virtual-environments](https://github.com/actions/virtual-environments) で Issue を開いてください。 このリポジトリには、ランナーに関するすべての主要なソフトウェア更新に関するお知らせも含まれています。

### Installing additional software

You can install additional software on {% data variables.product.prodname_dotcom %}-hosted runners. For more information, see "[Customizing GitHub-hosted runners](/actions/using-github-hosted-runners/customizing-github-hosted-runners)".

## Cloud hosts used by {% data variables.product.prodname_dotcom %}-hosted runners

{% data variables.product.prodname_dotcom %} hosts Linux and Windows runners on `Standard_DS2_v2` virtual machines in Microsoft Azure with the {% data variables.product.prodname_actions %} runner application installed. {% data variables.product.prodname_dotcom %}ホストランナーアプリケーションは、Azure Pipelines Agentのフォークです。 インバウンドのICMPパケットはすべてのAzure仮想マシンでブロックされるので、pingやtracerouteコマンドは動作しないでしょう。 For more information about the `Standard_DS2_v2` resources, see "[Dv2 and DSv2-series](https://docs.microsoft.com/azure/virtual-machines/dv2-dsv2-series#dsv2-series)" in the Microsoft Azure documentation.

{% data variables.product.prodname_dotcom %}は、{% data variables.product.prodname_dotcom %}自身macOS Cloud内でmacOSランナーをホストします。

## Workflow continuity

{% data reusables.actions.runner-workflow-continuity %}

In addition, if the workflow run has been successfully queued, but has not been processed by a {% data variables.product.prodname_dotcom %}-hosted runner within 45 minutes, then the queued workflow run is discarded.

## Administrative privileges

LinuxおよびmacOSの仮想環境は、パスワード不要の`sudo`により動作します。 現在のユーザが持っているよりも高い権限が求められるコマンドやインストールツールを実行する必要がある場合は、パスワードを入力する必要なく、`sudo`を使うことができます。 詳しい情報については、「[Sudo Manual](https://www.sudo.ws/man/1.8.27/sudo.man.html)」を参照してください。

Windowsの仮想マシンは、ユーザアカウント制御（UAC）が無効化されて管理者として動作するように設定されています。 For more information, see "[How User Account Control works](https://docs.microsoft.com/windows/security/identity-protection/user-account-control/how-user-account-control-works)" in the Windows documentation.

## IP アドレス

{% note %}

**ノート:** {% data variables.product.prodname_dotcom %}のOrganizationもしくはEnterpriseアカウントでIPアドレスの許可リストを使っているなら、{% data variables.product.prodname_dotcom %}ホストランナーは利用できず、代わりにセルフホストランナーを使わなければなりません。 詳しい情報については「[セルフホストランナーについて](/actions/hosting-your-own-runners/about-self-hosted-runners)」を参照してください。

{% endnote %}

To get a list of IP address ranges that {% data variables.product.prodname_actions %} uses for {% data variables.product.prodname_dotcom %}-hosted runners, you can use the {% data variables.product.prodname_dotcom %} REST API. 詳しい情報については「[GitHubメタ情報の取得](/rest/reference/meta#get-github-meta-information)」エンドポイントのレスポンス中の`actions`キーを参照してください。

Windows及びUbuntuのランナーはAzureでホストされており、そのためAzureのデータセンターと同じIPアドレスの範囲を持ちます。 macOSランナーは{% data variables.product.prodname_dotcom %}独自のmacOSクラウドでホストされます。

Since there are so many IP address ranges for {% data variables.product.prodname_dotcom %}-hosted runners, we do not recommend that you use these as allow-lists for your internal resources.

このAPIが返す{% data variables.product.prodname_actions %}のIPアドレスのリストは、週に1回更新されます。

## ファイルシステム

{% data variables.product.prodname_dotcom %}は、仮想マシン上の特定のディレクトリでアクションとシェルコマンドを実行します。 仮想マシン上のファイルパスは静的なものではありません。 `home`、`workspace`、`workflow` ディレクトリのファイルパスを構築するには、{% data variables.product.prodname_dotcom %}が提供している環境変数を使用してください。

| ディレクトリ                | 環境変数                | 説明                                                                                                                                |
| --------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `home`                | `HOME`              | ユーザ関連のデータが含まれます。 たとえば、このディレクトリにはログイン試行からの認証情報を含めることができます。                                                                         |
| `workspace`           | `GITHUB_WORKSPACE`  | アクションとシェルコマンドはこのディレクトリで実行されます。 このディレクトリの内容は、アクションによって変更することができ、後続のアクションでアクセスできます。                                                 |
| `workflow/event.json` | `GITHUB_EVENT_PATH` | ワークフローをトリガーしたwebhookイベントの`POST`ペイロード。 {% data variables.product.prodname_dotcom %}は、アクションを実行するたびにアクション間でファイルの内容を隔離するためにこれを書き換えます。 |

各ワークフローに対して{% data variables.product.prodname_dotcom %}が作成する環境変数のリストについては、「[環境変数の利用](/github/automating-your-workflow-with-github-actions/using-environment-variables)」を参照してください。

### Dockerコンテナのファイルシステム

Dockerコンテナで実行されるアクションには、 `/github`パスの下に静的なディレクトリがあります。 ただし、Dockerコンテナ内のファイルパスを構築するには、デフォルトの環境変数を使用することを強くお勧めします。

{% data variables.product.prodname_dotcom %}は、`/github`パス接頭辞を予約し、アクションのために3つのディレクトリを作成します。

- `/github/home`
- `/github/workspace` - {% data reusables.repositories.action-root-user-required %}
- `/github/workflow`

## 参考リンク
- 「[{% data variables.product.prodname_actions %} の支払いを管理する](/billing/managing-billing-for-github-actions)」
- You can use a matrix strategy to run your jobs on multiple images. For more information, see "[Using a matrix for your jobs](/actions/using-jobs/using-a-matrix-for-your-jobs)."

{% endif %}
