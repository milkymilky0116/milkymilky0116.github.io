---
title: Vim과 사랑에 빠지다 - Vim 입문기
date: 2024-01-25 01:03:43 +/-TTTT
categories: [vim]
tags: [vim]
mermaid: true # TAG names should always be lowercase
---

이번 글은 조금 가벼운 주제로 시작해 볼까 합니다. 여러분은 어떤 IDE 환경에서 개발을 하고 계시나요? 대부분은 VScode나 Intellij를 사용하고 계실 겁니다. 저도 개발을 시작한 이례로 쭉 몇 년 동안 VScode를 사용해 왔었고, VSCode가 세상에서 제일 좋은 IDE라고 굳게 믿고 있었습니다.

물론 IDE 환경을 전환했다고 해서 VScode에 대해 부정적으로 생각하는 것은 아닙니다. 여전히 VScode는 좋은 에디터라고 생각합니다. 무료이고, 처음 개발하는 사람도 금방 적응할 수 있을 정도로 사용법도 쉽고, 수많은 커뮤니티를 기반으로 제공되는 Extension Marketplace도 개발자로서의 삶에 많은 편안함을 제공합니다. 그런데 왜 VScode에서 하필 Vim으로 IDE 전환을 한 걸까요? Vim은 조작법도 어렵고, 요즘의 마법 같은 모던 IDE에 비해 기능도 부실해 보이는데 말이에요. Vim은 단순히 SSH 터미널 에디터 아니였나요?

![vscode.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/vscode.png?raw=true)
_이전까지의 저의 VScode 환경_

우선 VScode에서 Vim으로 전환하고 싶었던 이유들부터 하나씩 살펴보자면...

### 1. 생산성의 증대 (Vim motion)

VScode를 사용하면서 여느날 처럼 열심히 코딩을 하던 어느날, 유튜브의 알고리즘에 이런 영상을 보게 되었습니다.

{% include embed/youtube.html id='FrMRyXtiJkc' %}

이 영상을 본 저는 충격에 빠졌습니다. 첫 번째로 마우스에 손을 올리지 않고도 모든 태스크를 수행할 수 있다는 점이었고, 두 번째는 엄청나게 빠른 속도로 코드를 짤 수 있다는 점이었고, 세 번째는 모던 IDE의 기능 (Linting, Autocomplete...) 들이 모두 지원된다는 것이었습니다.

제가 가장 매료된 부분은 Vim motion이라고 일컬어지는 Vim의 커맨드 부분이었습니다. 단축키 몇 개로 파일을 빠르게 왔다 갔다 하고, 텍스트 에디팅을 매우 빠르게 수행할 수 있으며, 무엇보다 마우스가 필요 없다는 것이었지요. 저는 영상을 보고 바로 결심이 들었습니다. **이건 꼭 배워야겠다.**

물론 VScode에도 Vim 관련 플러그인이 존재합니다. [vscodevim](https://github.com/VSCodeVim/Vim) 이라는 익스텐션으로 VScode에서도 Vim motion을 사용할 수 있지요. Intellij에도 [ideavim](https://github.com/JetBrains/ideavim) 이라는 플러그인이 존재합니다. 그렇다면 굳이 IDE 환경을 바꿔야 하나 라는 생각이 들 법도 하지만..

### 2. 마음대로 커스터마이징 가능한 나만의 IDE

![vim.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/vim.png?raw=true)
_현재 저의 Vim(Neovim) 환경_

영상을 보고 난 직후 단순한 터미널 에디터인 줄만 알았던 Vim이 모던 IDE의 기능을 제공한다는 것이 신기해서 서칭을 하기 시작했습니다. 조금 찾아보니, 기존의 Vim에서 더 업그레이드된 [Neovim](https://neovim.io/)이 그러한 기능을 제공한다는 것을 알게 되었습니다. 그런데 이거, 점점 더 알아보니 단순히 테마 설정이나 Autocomplete 설정을 넘어서 모든 것을 커스터마이징 할 수 있다는 사실을 알게 되었습니다. 정말 **말 그대로** 모든 부분을요.

LSP라고 불리는 언어 지원 플러그인은 기본이고, 파일 트리 또한 원한다면 플러그인으로 넣을 수 있습니다. 파일을 빠르게 탐색할 수 있는 플러그인부터, 탭 라인 설정, Code Action 단축키 설정까지.. 나에게 필요한 기능들만 넣은 나만의 IDE를 만들 수 있는 것입니다. Neovim 플러그인 생태계는 생각한 것보다 훨씬 커서, VScode에 이런 게 왜 없었을까 싶은 플러그인도 많았습니다. 이 모든 설정을 Lua라고 불리는 프로그래밍 언어로 꾸밀 수 있는데, 비록 생소하지만 언어 자체가 매우 심플해서 누구나 조금만 코드를 들여다본다면 쉽게 파악이 가능합니다.

저한테 가장 큰 메리트는, 설정 자체를 Git으로 백업하고 버전 관리할 수 있다는 점이었어요. 저는 집에서는 데스크탑 밖에서는 노트북을 들고 다니는데요, 항상 데스크톱의 VScode 설정들을 그대로 노트북에 불러올 수 없다는 점이 많이 아쉬웠어요. 하지만 Neovim에서는? 업데이트한 설정을 git에 푸시 해놓고 노트북에서 단순히 pull 하면 1초 만에 데스크탑의 업데이트된 환경을 그대로 불러올 수 있었습니다. 빠르고 간편하죠.

### 3. Vim을 사용하고 싶은 이유는 Vim을 사용해보고 싶고 난 원하니까.

![쿨펀섹.gif](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/%EC%BF%A8%ED%8E%80%EC%84%B9.gif?raw=true)

(끄떡.)

### 단점 - 어어어어엄청나게 큰 러닝커브

![vim_meme2.jpeg](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/vim_meme2.jpeg?raw=true)
_Vim 어떻게 나가요..? 에 4000개 좋아요가 달린 모습_

이렇게 좋은 IDE가 될 수 있는데 Vim이 왜 세상을 지배하지 않았을까요? 터미널에서 Vim으로 텍스트 에디팅을 해본 사람이라면 알 테지만, 아무것도 모른 상태에서 Vim을 켜면 기존과는 다른 조작체계에 혼란스러워집니다. 커서 조작은 기본적으로 `hijk`로 움직이고, 복사 붙여넣기는 `Ctrl-c` `Ctrl-v` 가 아니라, 한 행 복사는 `yy` 붙여넣기는 `p`입니다. 저장은 `Ctrl-s` 가 아니라 `:w`이고, vim을 나가는 것은 `:q`입니다. 세상에. 단순한 텍스트 에디팅 단축 키들인데 벌써 포기하고 싶어집니다. 그뿐만일까요, Vim에는 Normal, Insert, Visual이라고 하는 크게 세 가지 모드가 존재하고 각각의 모드에는 수많은 단축키들이 존재합니다. Vim의 모드와 단축키 구조들을 다이어그램으로 그려보면 이렇습니다...

![vimModeStateDiagram.svg](https://raw.githubusercontent.com/milkymilky0116/obsidian-backup/3dbd4c3cb3bbac1dfef93767bda23334421651ed/image/vim/vimModeStateDiagram.svg)
_정신이 아득해지는...._

맞습니다. Vim은 러닝 커브가 높은 텍스트 에디터입니다. 수많은 단축키 조합이 존재하고 이 많은 것을 한 번에 숙달하기란 쉽지 않은 일입니다. 저도 Vim을 처음 일주일 동안 사용하면서 VScode가 많이 그리웠고, 생산성을 늘리고 싶어서 선택했는데 되려 느려지기까지 했으니까요. 이런 단축키들에 익숙해지지 않은 채로 코딩을 해야 한다면 정말 괴로운 시간이 될 수도 있습니다.

그럼에도 불구하고 생산성을 늘리고 싶고, Vim을 정말 사용하고 싶다면 몇 가지 공유해 드리고 싶은 팁들이 있습니다.

1. 바로 IDE 환경을 바꾸지 마세요.

Vscode나 Intellij에서 바로 Vim으로 넘어가는 것은 앞에서도 말씀드렸듯 어려운 일입니다. 그래서 Vim으로 바로 넘어가는 것은 추천드리지 않습니다. 업무 능률이 떨어질 수도 있고, Vim에 흥미를 잃기 쉽기 때문입니다. 가장 편안한 환경에서 계속해서 코딩을 하는 것이 현명한 생각입니다. 그렇다면 Vim의 커서 이동이나 단축키에 익숙해지고 싶으면 어떻게 해야 할까요?

앞서 언급했듯 Vscode와 Intellij에는 각각 [vscodevim](https://github.com/VSCodeVim/Vim),[ideaVim](https://github.com/JetBrains/ideavim)이라는 플러그인이 존재합니다. 여러분이 가장 익숙한 환경에서 Vim motion을 사용할 수 있게끔 지원하는 플러그인들이지요. 이 플러그인들을 항상 켜 놓는 게 아니라, 하루에 10분 정도만 시간을 투자하는 것입니다. `hjkl`로 커서 이동을 하는 것에 익숙해지고, 단축키를 하루에 하나씩만이라도 숙지하다 보면 어느새 금방 Vim motion에 익숙해질 겁니다.

2. 좀 더 재밌는 방법으로 Vim에 익숙해지기

Vim motion을 바로 텍스트 에디팅에 적용하고, 수많은 단축키와 커맨드를 외우는 일은 지루할 수 있습니다. 하지만 Vim motion은 외우는게 아니라 헬스나 운동같이 머슬 메모리의 영역이라고 개인적으로는 생각합니다. 그래서 바로 코딩을 하면서 익히기 보다는, 좀 더 재밌는 방법으로 Vim motion을 익히게 되면 동기도 생기고 생각보다 꽤 할만하다는 것을 알 수 있을겁니다.

- [Vim Adventure](https://vim-adventures.com/)

![vim_adventure.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/vim_adventure.png?raw=true)

Vim Adventure라고 하는 웹 게임입니다. 기본적인 커서 이동부터 여러 가지 Vim motion들을 Level 별로 진행하면서 익힐 수 있습니다. 단점은 Level 2부터는 유료라는 점이에요. Vim에 정말 익숙해지고 싶고 좀 더 재밌는 방법으로 배우고 싶은 분들에게 추천드립니다.

- Vimtutor

![vimtutor.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/vimtutor.png?raw=true)

Vim에 기본 내장되어 있는 튜토리얼입니다. Mac/Linux 사용자는 `vimtutor`라고 터미널에 입력하면 튜토리얼이 나오게 됩니다. 저는 시간이 날 때 틈틈이 챕터 하나씩 연습해 보면서 Vim을 익혔습니다. 따로 설치 과정도 필요 없고, 챕터별로 차근차근 씩 따라가는 구조로 되어있기 때문에 가장 추천하는 방법 중 하나에요.

- [ThePrimeagen/Vim-be-good](https://github.com/ThePrimeagen/vim-be-good)

![vim-be-good.png](https://github.com/milkymilky0116/obsidian-backup/blob/main/image/vim/vim-be-good.png?raw=true)

VimBeGood은 Neovim 플러그인으로, 커서 이동이나 vim motion 들을 게임같이 배울 수 있다는 장점이 있습니다. 난이도 조절도 가능하고, 나름대로 시간 기록을 깨면서 달성하는 성취감도 얻을 수 있습니다. 단점은 Neovim 플러그인이라서 Neovim을 세팅하는 법을 어느 정도 알아야 한다는 점이에요.

### Neovim 세팅하기

자, 이제 Vim motion에 제법 익숙해졌고 Vim을 정말로 IDE로 사용해 보고 싶습니다. 기존의 모던 IDE 들의 기능을 가져가면서 Neovim을 세팅하려면 어떻게 해야 할까요?

먼저 미리 말씀드려야 할 점은, Neovim의 세계는 정말 정말 방대하다는 것입니다. 여러분이 원하는 대로 커스터마이징이 가능하기 때문에, 반대로 말하자면 수많은 옵션이 존재하고 어떻게 세팅하는지 일일이 찾아봐야 한다는 점이지요. 그래서 이 글에서 완벽하게 Neovim을 어떻게 세팅해야 하는지 가이드를 해 드릴 수는 없습니다. 하지만 유튜브에 조금만 검색해 보면 수많은 튜토리얼들이 존재하므로, 한번 둘러보고 나에게 맞는 세팅법을 찾는 것이 가장 좋은 방법입니다.

Neovim을 세팅하는 방법에는 크게 두 가지 방법이 존재합니다. 하나는 말 그대로 맨땅에서부터 세팅하는 법과, Neovim Distro라고 불리는 미리 설정되어 있는 Config를 가져오는 것입니다.

### Neovim Distros?

![nvchad.webp](https://raw.githubusercontent.com/milkymilky0116/obsidian-backup/main/image/vim/nvchad.webp)

Neovim Distro는 앞에서도 설명 드렸듯 미리 설정되어 있는 Config들을 의미합니다. 파일트리, 언어지원, 탭 등등 모던 IDE에 익숙해진 개발자들이 바로 사용할 수 있게끔 환경을 미리 세팅했다고 알고 계시면 될 것 같습니다.

Neovim Distro에는 여러가지 종류가 존재합니다. 그 중에서 가장 추천드리는 세팅은 다음과 같습니다.

- NVChad

{% include embed/youtube.html id='Mtgo-nP_r8Y' %}

이름과 썸네일부터 범상치 않은 이 Distro는 초보자들이 많이 쓰는 세팅입니다. 저도 잠깐 써봤지만 확실히 모던 IDE에 익숙해져 있는 사람이 바로 쓰기에 좋은 세팅으로 되어 있지 않나 라고 생각합니다. 단점은 플러그인 설치가 조금 어렵다는 점입니다.

- Lazyvim

{% include embed/youtube.html id='N93cTbtLCIM' %}

제가 현재 사용하고 있는 Distro 입니다. Lazyvim의 가장 큰 장점은 Neovim의 생태계를 알아가기 좋게 적당히 미니멀리즘으로 구성되어 있다는 점입니다. 플러그인 설치도 쉽고, 단축키 설정이나 환경설정을 쉽게 적용 할 수 있어서 나만의 IDE 환경을 꾸려 나가기에 최적화 되어 있습니다.

- [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)
  `kickstart.nvim`은 가장 미니멀리즘한 Distro 입니다. 기본적인 세팅만 되어있고 IDE를 어떻게 세팅할 것인지는 여러분의 손에 달려 있습니다. 그렇기 때문에 다른 Distro를 사용하면서 Neovim 생태계를 익히다가, 커스터마이징을 좀 더 세세하게 해보고 싶을때 사용하면 좋을 것 같습니다. 저도 Lazyvim을 사용하고 있지만 `kickstart.nvim` 으로 천천히 옮겨볼 생각이에요.

### 아예 처음부터 세팅해보기?

{% include embed/youtube.html id='w7i4amO_zaE' %}

물론 처음부터 세팅하는 것도 좋은 생각일 수 있습니다. 유튜브에 플러그인 매니저부터 설치하는 친절한 가이드도 많이 나와있고, 불가능 한 일은 아닙니다. 하지만 Neovim 생태계에 익숙지 않은 상태에서 가이드를 따라 해보니 너무 어려운 점이 많았습니다. Lua라는 언어도 익숙지 않은데 이 언어를 가지고 이것저것 플러그인 세팅과 키 매핑을 일일이 해야 한다는 게 처음에는 너무 부담이 클 수 있습니다. 그래서 앞서 말씀드렸듯 Neovim의 세계에 조금씩 적응하면서 미니멀리즘으로 가는 방향이 저는 개인적으로 맞다고 생각합니다. 저도 천천히 더 미니멀리즘적인 세팅으로 옮겨가볼 생각입니다.

### 그래서 한달동안 Vim을 써본 소감이 어떠냐면...

**...너무 좋았습니다.** 처음에는 물론 러닝커브가 있는 편이였어요. 하지만 vim motion과 단축키에 점점 익숙해지면서 더 빠른 속도로 생산성 있게 일을 수행하는게 가능했어요. 파일을 빠르게 옮겨다니고, vim motion을 이용하면서 효율적으로 코딩하는게 너무 만족스러웠습니다. 특히나 저는 Code Action을 제가 원하는 단축키에 할당해서 편하게 작업하는게 좋았습니다. 귀찮은 작업을 빠르게 넘길 수도 있고, 에러 진단도 쉽게 볼 수 있고, 심지어 자동으로 테스트를 생성해주기 까지하거든요. 이런 작업들을 직접 커스터마이징 해줄 수 있다는 점이 저에게는 큰 매력 포인트였습니다. Neovim이 모던 IDE에 못지 않은 방대한 생태계를 가지고 있는 덕택에, 빠른 생산성을 가져 가면서도 모던 IDE의 기능들을 동시에 쓸 수 있어서 더 생산적으로 일을 할 수 있게 된 것 같습니다.

만약 더 생산적으로 코딩을 해보고 싶고 직접 IDE 환경을 꾸려 나가는 것에 관심이 많다면, Vim을 한번 시도해보는 건 어떨까요? 긴 글 읽어주셔서 감사합니다 😊
