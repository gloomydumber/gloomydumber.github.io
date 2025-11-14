+++
title = "Vocs Quick Look through a x402 Docs Clone"
date = 2025-11-10
[extra]
toc = true
comment = true
+++

wevm 팀이 개발한 문서화 프레임워크인 `vocs`가 있다. 이 글은 이미 공개된 x402 문서를 개인적으로 vocs로 다시 작성해 보면서, 그 과정에서 vocs의 기능을 탐색해 본 내용을 다룬다. vocs를 이용하여 작성한 문서는 [Github Pages](https://gloomydumber.github.io/vocs-coinbase-x402-documentation-clone/)에 배포하여 확인할 수 있다.

## wevm

[wevm](https://wevm.dev/) 이라는 이더리움을 위한 TypeScript 툴(Tool)을 개발하는 팀이있다.
wevm 팀에서는 [`viem`](https://viem.sh/)이나 [`wagmi`](https://wagmi.sh/)처럼 블록체인 개발에서 자주 사용되는 TypeScript 기반으로 작성된 라이브러리나 프레임워크 형태의 몇몇 오픈소스 프로덕트를 개발하고 유지 보수하고 있다. wevm 팀이 개발하고 관리하는 프로덕트를 간단히 설명하자면:

- `viem`은 기존에 `web3.js`나 `ethers.js`로 작성되던 EVM 상호작용 인터페이스 라이브러리를 TypeScript 기반으로 쓸 수 있도록 기능을 제공한다.
- `wagmi`는 TypeScript 기반으로, 지갑 연결 기능과 같은 다양한 블록체인 관련 기능을 다루는 여러 React Hook을 제공한다.
- [`Ox(⦻)`](https://oxlib.sh/)는 TypeScript 기반의 Ethereum Standard Library이다. EVM과 상호작용하는 가장 기초적인 기능만을 구현했다. unopinionated Standard Library의 개념으로 단독으로 사용되기 보다는 `viem` 이나 [`Tevm`](https://tevm.sh/) 내부에서 활용된다.
- [`vocs`](https://vocs.dev/)는 지금 이 포스트에서 주로 다루고자 한다.

wevm 팀이 개발하는 이런 여러 TypeScript 기반 프로덕트는 블록체인 리서치 VC로 유명한 [Paradigm](https://www.paradigm.xyz/)의 후원을 통해 진행되고 있다.
이더리움의 Rust 버전 클라이언트로 유명한 [reth](https://reth.rs/) 개발과 Smart Contract 개발 프레임워크로 유명한 [Foundry](https://getfoundry.sh/) 개발도 Paradigm에서 주도했는데, Paradigm이 TypeScript 기반의 블록체인 개발 환경에도 상당히 기여하고 있는 셈이다.

## vocs

본문에서 주로 다룰 `vocs`는 wevm 팀에서 TypeScript 기반으로 React와 Vite로 동작하는 문서화 프레임워크다.

wevm 팀의 `viem`, Paradigm 주도 개발 프로젝트인 `reth`, `Foundry`, `alloy`도 문서화 프레임워크로 vocs를 사용했다. wevm 팀이나 Paradigm과 관련없는 vocs를 사용한 기타 프로젝트로는 랜딩 프로토콜인 Morpho가 있는 것으로 확인했다. 아무래도 블록체인과 관련된 프로덕트가 vocs를 문서화 프레임워크로 사용할 가능성이 있는 것 같다.

블록체인과 관련되지 않은 경우에 vocs를 사용한 유명한 프로덕트의 경우는, SSG나 문서화 프레임워크가 많다보니 잘 모르겠는데 아마 높은 확률로 없는 것 같다.

특이 사항으로는 wevm 팀의 [wagmi](https://wagmi.sh/)의 경우, [구버전 문서](https://1.x.wagmi.sh/)에는 [`nextra`](https://nextra.site/)를 사용했고, 현재 버전으로는 [`vitePress`](https://vitepress.dev/)를 이용하고 있는 것으로 확인했다. viem의 경우도 기존에 vitePress를 사용했는데, wagmi는 무슨 이유에서인지 아직 vocs로 마이그레이션하진 않았다.

요약하자면 `vocs`를 문서화 프레임워크로 사용하는 프로덕트의 예는 다음과 같다:

- [vocs](https://vocs.dev/) (당연하지만 vocs 문서도 vocs로 빌드되었다)
- [viem](https://viem.sh/)
- [reth](https://reth.rs/)
- [Foundry](https://getfoundry.sh/) (최근 문서가 [mdBook](https://github.com/rust-lang/mdBook)을 이용한 Foundry Book 개념에서 vocs로 빌드 하는 것으로 변경된 듯하다)
- [alloy](https://alloy.rs/)
- [Morpho](https://docs.morpho.org/get-started/)
- 기타 내가 모르는 프로덕트지만 높은 확률로 블록체인과 관련된 프로덕트

아무튼, `vocs`라는 문서화 프레임워크를 보자마자 이건 한 번 간단히라도 사용해보고 싶다는 생각이 굴뚝같이 들었는데 그 이유는:

- `web3.js`나 `ethers.js`를 사용하면서 느끼던 타입에 대한 갈증을 `viem`을 통해 시원하게 해갈한 경험이 있었다. 그 `viem`을 개발한 wevm 팀이 vocs를 개발했고, `viem`의 문서도 vocs를 통해 생성되었다.
- Paradigm이 wevm 팀과 vocs를 후원한다.
  - Paradigm은 개인적으로 매력을 느끼는 언어인 Rust 언어로 작성 된 이더리움 클라이언트 `reth`를 개발하고 유지 보수하여 이더리움 클라이언트 다양성 확보에 기여하고 있다.
  - Paradigm은 이제는 [`hardhat`](https://hardhat.org/)보다 대중적으로 쓰이는 Smart Contract 개발 프레임워크인 `Foundry`를 개발하고 유지 보수하고 있다.

이렇듯 vocs는 하이프있고 개발자 친화적이면서 심지어 직접 개발도 하는 리서치 VC에서 후원하고 있다. 이런 이유로만 vocs를 사용해보고자 마음먹은게 Hype-Driven-Development 내지는 Hype-Driven-Documentation은 아니라고 자신할 수 있다. wevm 팀과 Paradigm 에서 제공하는 다른 프로덕트를 사용해본 경험(viem, Foundry)이 실제로 좋았기 때문이다.

## x402

vocs의 기능을 알아보기 위해 무엇을 문서화 해볼까 고민하다가, 최근에 화제가 되는 온체인 결제 관련 내러티브인 [**x402**](https://x402.org) 문서를 clone-documenting 해보기로 했다.

x402는 Coinbase 주도로 개발된 온체인 결제 프로토콜로, HTTP 402 상태 코드를 활용한다. [HTTP 402 Payments Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/402) 상태 코드는 수 십년 간 활용되지 않던 것인데, 이를 활용한다는 것이 특징이다. x402 문서를 vocs로 clone-documenting하면서 기존 영문 문서를 번역하여 다루어 보았기 때문에 이 포스트에서는 x402에 대한 자세한 설명은 줄인다. vocs를 이용하여 빌드하고 [github pages](https://gloomydumber.github.io/vocs-coinbase-x402-documentation-clone)에 배포하는 등의 과정에서 x402에 대해 조사하고 명세를 번역하고 윤문하였다.

x402는 Coinbase에서 주도적으로 개발되었지만 대중에 공개된 오픈 프로토콜이기도 해서, 두 개의 문서가 존재한다.

- [x402.org 문서](https://x402.gitbook.io/x402) ([x402.org](https://www.x402.org/)라는 x402 오픈 프로토콜 랜딩 페이지에서 제공됨)
- [Coinbase Developer Platform 문서 | x402](https://docs.cdp.coinbase.com/x402/welcome)

`x402.org`의 문서는 커뮤니티에서 유지 보수되는 문서이고 [`GitBook`](https://www.gitbook.com/)을 통해 작성되었다. 반면에 `Coinbase Developer Platform` 문서는 Coinbase 측에서 관리되고, [`Mintlify`](https://www.mintlify.com/)를 통해 작성되었다.

아무래도 커뮤니티에서 관리되는 GitBook으로 작성된 문서는 무료인 Basic-tier 플랜으로 작성되었기 때문에 주목할만한 기능적인 부분이 없어서 Mintlify로 작성된 Coinbase의 문서를 기준으로 vocs를 이용한 clone-documenting을 진행하였다. 예외로 문서의 랜딩 페이지는 오픈소스의 것을 clone-coding하였다.

## Features

vocs의 기능에 대해 먼저 알아보았다. 사실 역시 여느 문서화 프레임워크나 SSG(Static Site Generator)와 크게 다르지 않다. 아래와 같은 [vocs 공식 문서](https://vocs.dev/docs)의 목차를 기준으로 어떤 기능이 있는지 대략 서술하면:

![vocs.dev_docs](img/vocs.dev_docs.png)

- **Getting Started**: Vite로 빌드되는 React 기반 문서 프레임워크로서 Markdown 및 MDX를 지원하는 것을 설명하고, 설치 방법에 대해 서술하고 있다. 설치 방법으로는 `npm` 과 같은 패키지 라이브러리나 `Vercel`을 통해 부트스트랩하거나, 직접 수동 설치하는 법에 대해 설명하고 있다.
- **Project Structure**: 프로젝트의 전체적인 폴더 구조와 설정 파일, 라우팅, Static Assets의 경로 등에 대해 설명한다.
- **Markdown Reference**: 기본적인 마크다운 문법을 포함해 Callout 기능이나 코드 블록을 탭으로 엮은 Code Groups 기능 등을 설명한다. wevm 팀 자체가 TypeScirpt 툴을 개발하는 팀임을 고려하면, [TypeScript의 TwoSlash](https://www.typescriptlang.org/dev/twoslash/) 지원하는 것이 가장 큰 특징이다. TwoSlash 기능은 TypeScript에서의 오류 등을 코드 블록에 표기할 수 있는 문법을 제공한다.

![twoslash](img/twoslash.png)

### 가이드

- **Blog**: 기본적으로 문서화 프레임워크지만 블로그 아티클을 작성할 수 있는 기능도 있다.
- **Code Snippets**: 코드 블록을 작성할 때 Code Snippets 기능을 이용할 수 있는데, 이에 대해 설명한다. Code Snippets 기능은 코드 블록에서 다른 코드를 임포트하는 등의 기능을 제공한다.
- **Components**: MDX에서 사용할 수 있는 기본적으로 제공하는 컴포넌트에 대해 명세하고, 커스텀 컴포넌트를 작성하는 방법에 대해 설명한다. Vite로 빌드되기 때문에 `MUI` 등의 외부 라이브러리나 프레임워크를 사용해서 컴포넌트를 꾸밀 수 있다.
- **CSS & Styling**: 커스텀 CSS를 통해 스타일링 하는 방법에 대해 설명한다.
- **Dynamic OG Images**: Open Graph Image를 설정하는 기능에 대해 설명한다. 카카오톡, 디스코드, 텔레그렘 등에 채팅에 URL을 포함하면 자동으로 생성되는 이미지가 Open Graph Image다.
- **Layouts**: MDX의 Frontmatter로 Layout 값을 설정할 수 있는데 이에 관해 설명하고 있다. `docs`로 설정하면 일반적인 문서 페이지고, `landing` 으로 설정하면 랜딩 페이지로 작성할 수 있다.
- **Markdown Snippets**: Markdown Snippets를 작성하고 임포트할 수 있는 기능에 대해 설명하고 있다.
- **Sidebar & Top Navigation**: 문서 목차를 보여주는 사이드바와 문서의 구성 페이지를 보여주는 탑 네비게이션에 대한 기능과 설정값을 설명한다.
- **Theming**: 테마에 대한 설정을 설명한다.
- **Twoslash**: TypeScript에서의 오류 등을 코드 블록에 표기할 수 있는 문법인 Twoslash 기능에 대해 설명한다.

### API

- **Config**: 설정 파일인 `vocs.config.ts` 파일에 설정할 수 있는 여러 값과 기능을 설명한다. `remark`와 `rehpye` 플러그인 설정도 자유롭게 가능하다.
- **Frontmatter**: 마크다운과 MDX 파일의 Frontmatter를 통해 설정할 수 있는 여러 값과 기능을 설명한다.

요약하자면, 대략 일반적인 문서화 프레임워크의 기능을 갖추었고, TypeScript의 오류를 코드 블록에 표기할 수 있는 Twoslash 문법을 적용할 수 있는 기능이 다른 문서화 프레임워크와 특히 차별화된다. TypeScript로 작성되는 프로젝트라면 vocs를 통해 작성해보는 것도 좋아보인다.

## Translation

vocs의 위와 같은 기능을 바탕으로, 여러 설정을 하고 난뒤에 x402 문서를 번역하여 옮겨적었다. 몇 번 해봤지만 아직 번역 내지는 의역해서 자연스럽게 작성하는 것이 어려운 것 같다.

번역 과정에서 '_요청-응답_', '_헤더_', '_페이로드_'와 같은 일반적인 기술 용어는 큰 문제가 되지 않는데, x402 프로토콜에서만 사용되는 용어인 '_Facillitator_' 나, 결제와 정산 등을 표현할 때 쓰이는 '_Payment_', '_Settlement_'와 같은 용어는 신경써서 번역해야 했다.

특히, _Facillitator_ 는 직역하면 _조정자, 중재자, 촉진자, 조력자_ 등 서로 비슷하면서도 다른 의미로 해석될 가능성이 많고, x402 프로토콜 자체에서 쓰이는 기술 용어(terminology)에 가깝다고 생각해 일단 '퍼실리테이터'라고 썼다.

_Facillitator_ 는 간략하게 정의하자면, 결제 과정에서 구매자와 판매자 간의 결제가 제대로 이루어지는지 온체인 검증과 정산을 돕는 주체이다. 이를 직역해서 _조정자, 중재자, 촉진자, 조력자_ 라고 번역하기에는 혼란스러울 것 같았다.

'_Payment_' 와 '_Settlement_' 도 대충 번역하면 상당히 곤란하다. 일반적으로 '_Payment_' 가 이루어졌다고하면 지불과 결제가 모두 이루어졌다고 착각할 수 있는데, 지불과 결제는 별도의 과정이다. 회계 용어로서 '_Payment_'는 지급이라는 뜻이고, '_Settlement_'가 결제라는 뜻이다. 이를 합쳐 '지급결제 (Payment Settlement)'라고 한다. ([한국은행 지급결제제도 개념](https://www.bok.or.kr/portal/main/contents.do?menuNo=200345))

특히, 해당 문서에서 다루는 블록체인에서의 '_Settlement_' 는 트랜잭션의 Confirmation이 성공적으로 이루어져서 결제가 성공적으로 완료되었다는 의미를 담고 있기도 하다. 따라서 x402가 결제 과정을 다루는 프로토콜인만큼 사용되는 용어가 기술적 맥락을 고려하면서도 회계-경제 용어임을 고려해서 번역해야했다.

## Custom Component

MDX를 이용하여 커스텀 컴포넌트를 써야하는 경우가 있었다. Coinbase x402 문서 중 한 부분을 보면 아래와 같은 카드 컴포넌트가 있다.

![cdpCard](img/cdpCard.png)

마우스를 hover하면 테두리가 강조되고, 클릭하면 같은 페이지에 존재하는 섹션으로 해시 라우팅된다. 이와 같은 기능을 하는 컴포넌트를 아래 이미지와 같이 [클론 코딩](https://github.com/gloomydumber/vocs-coinbase-x402-documentation-clone/blob/a40bf76352e90b61f6c048bb3639d84673fb9cb1/docs/components/FacillitatorModelCards.tsx#L66)했다. 컴포넌트 자체는 MUI를 활용했다. MUI의 Grid, Card 등을 사용하고 아이콘도 MUI에서 제공하는 것을 사용했다. 마우스 호버시 테두리 색상을 `var(--vocs-color_borderAccent)`로 설정하는 등 CSS의 경우 vocs에서 미리 정의된 CSS 변수값을 활용하여 문서의 기존 UI와 통일성을 해치지 않으려고 했다.

![vocsCard](img/vocsCard.png)

아래의 페이지에서 컴포넌트의 동작을 직접 비교할 수 있다:

- [Coinbase Developer Platform Docs | x402 - Network & Token Support](https://docs.cdp.coinbase.com/x402/network-support#overview)
- [내가 vocs로 작성하여 배포한 x402 문서](https://gloomydumber.github.io/vocs-coinbase-x402-documentation-clone/network-and-token-support/#%EA%B0%9C%EC%9A%94)

또, vocs에서는 문서의 랜딩 페이지도 MDX로 작성할 수 있다. Coinbase의 경우 별도의 x402 랜딩 페이지가 없지만, 오픈 프로토콜 문서의 경우에 랜딩 페이지가 있어서 그것을 클론 코딩했다.

아래의 페이지에서 직접 비교할 수 있다:

- [https://www.x402.org/](https://www.x402.org/)
- [내가 vocs로 작성하여 배포한 x402 문서](https://gloomydumber.github.io/vocs-coinbase-x402-documentation-clone/)

## Github Pages Deployment

배포는 여러 방법으로 할 수 있지만 Github Pages를 통해 해보았다.

지금 블로그의 CI/CD 방법과 마찬가지로 레포에 푸쉬가 이루어질 때마다 Github Pages로 배포되도록 아래와 같이 Github Workflows `.yml` 파일을 설정했다.

```yml
name: Build and deploy Vocs site

on:
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build Vocs site
        run: npm run build

      - name: Deploy to gh-pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs/dist
          publish_branch: gh-pages
```

처음에 `checkout` 과정을 `fetch-depth` 값 설정없이 했더니 이전 깃 커밋 내역을 캐싱하지 않고 빌드하여 빌드 할 때마다 문서의 모든 페이지의 최근 수정 시간이 동일하게 빌드 시점으로 갱신되는 문제가 있었다. vocs의 [vocs/src/vite/utilsgetGitTimestamp.ts](https://github.com/wevm/vocs/blob/c00edbe083ada9245da0d635c7735d74231818be/src/vite/utils/getGitTimestamp.ts#L7) 에 따르면 Git Timestamp 기반으로 문서 페이지의 _Last updated_ 항목을 표시해주는 것을 알 수 있다. `fetch-depth` 값을 0으로 제대로 주고나니, 정상적으로 동작하였다.

![lastUpdated](img/lastUpdated.png)

또 빌드 시에 겪은 다른 문제로는 설정 파일에서 인덱스 경로를 제대로 설정했음에도 불구하고 배포된 URL의 인덱스 지점이 `https://gloomydumber.github.io/vocs-coinbase-x402-documentation-clone/` 와 같이 URL의 하위 경로인 `/vocs-coinbase-x402-documentation-clone/` 경로 부분에 배포되었다보니 Static 파일을 임포트하거나 링크를 작성할 때 상대경로로 작성하니 오류가 발생해서 절대경로로 작성하였다.

## Issues

vocs를 이용해 문서를 작성해보면서 생긴 크고 작은 이슈를 정리해보았다.

### Windows 환경 미지원

Windows 환경에서 `npm init vocs` 명령어를 통해 프로젝트를 부트스트랩 하려고하면 동작하지 않는다. 디펜던시 중에 `\` 백슬래쉬를 사용하는 것이 있는 것 같은데 이것이 Windows 환경에서 경로를 처리하는데 문제를 발생시키는 것 같다. 해당 문제를 기술하는 [이슈](https://github.com/wevm/vocs/issues/52)가 있는데, 현재 `Closed as not planned` 상태이다. 해당 문제를 고치고자 시도한 [관련 PR](https://github.com/wevm/vocs/pull/128)이 존재하는데, 1년째 무시되고 PR 작성자는 결국 그냥 PR을 닫았다. PR을 닫은 날 달린 [프로젝트 관리자의 코멘트](https://github.com/wevm/vocs/pull/128#issuecomment-2686811801)가 가관인데 PR이 왜 머지되지 못했는지 부연 설명없이 WSL의 설치 링크만 남겼다.

![install_wsl](img/wsl.png)

약간 마음이 껶였지만 WSL을 통해 설치했다.

### Bold체 표기 문제

문서를 작성하면서 느꼈는데, `Bold(****, ___)`로 표기한 글자가 딱히 두껍다고 느껴지지 않았다. [코드(`vars.css.ts`)](https://github.com/wevm/vocs/blob/c00edbe083ada9245da0d635c7735d74231818be/src/app/styles/vars.css.ts#L377)를 확인해보니 기본적으로 프로젝트에서 설정한 볼드는 `vocs_Strong` 이라는 클래스명을 갖게 빌드되는데, 이 클래스의 CSS는 `font-weight` 값을 `--vocs-fontWeight_semibold` 라는 변수명으로 처리한다. 그런데 `--vocs-fontWeight_semibold`에 지정된 값이 겨우 `500` 이다. [mdn 문서](https://developer.mozilla.org/ko/docs/Web/CSS/Reference/Properties/font-weight#600)에 따르면 Semi Bold는 `600` 으로 표기되어 있다. 볼드치고 글자가 얇다고 느껴진 것이 느낌만 그런게 아니라 실제로 값이 낮게 설정되어있었다.

이는 의도한 바인지, 실수인지는 알 수 없다. Custom CSS 적용이 가능하니 일단은 사소한 문제이지만 신경 쓰이기는 한다.

### Blank Page

마찬가지로 사소한 문제인데, 공식 문서에 있는 문제라서 vocs를 이용해서 작성하는 문서에는 지장이 없는 문제이긴하다.

공식 문서에 [https://vocs.dev/blank](https://vocs.dev/blank) 라는 의미를 알 수 없는 페이지도 같이 배포되어있다.

테스트용으로 작성한 것 같은데, 문제는 vocs 공식 문서에 검색 엔진을 통해 뭔가를 검색할 때 이 페이지도 같이 인덱싱된다.

### ToC 버그

최신 버전에 페이지를 옮기면 새로운 페이지에 맞게 ToC가 변경되지 않는 버그가 있다. 문서의 다른 페이지로 들어가도 기존에 있던 페이지의 ToC가 그대로 표기된다.

viem 처럼 비교적 최신 버전을 쓰는 문서들은 해당 문제가 그대로 벌어지는 것을 확인했고, Foundry나 Morpho 처럼 vocs 버전을 업데이트해서 배포하지 않은 문서들은 문제가 없다.

해당 문제는 [이슈](https://github.com/wevm/vocs/issues/333)로 제출되어 있는 상태다.

### ToC 커스텀 기능 부재

`####(<h3>)` 태그까지는 ToC에 색인되어 표기되는데, `####`(`<h4>`) 태그부터는 표기가 되지 않는다. Mintlify로 작성된 Coinbase의 x402 문서에서는 ToC에 표기된 항목이 vocs로 옮겨 적을 때는 표기할 수 없었다.

어느 헤딩 넘버링까지 ToC에 표기할 것인지 별도의 설정이 있으면 좋을 것 같다.

### Table & Detail (Accordion) UI

여느 SSG가 대부분 그렇지만 기본적으로 제공하는 테이블과 디테일(아코디언)이 그렇게 예쁘지 않다.

별도로 컴포넌트를 통해 생성하거나 CSS 설정을 따로 하는 것이 좋을 것 같다.

이에 비해, Coinbase X402 문서에서 사용한 mintlify의 공식 문서에서 [Accordion](https://www.mintlify.com/docs/components/accordions)과 [Table](https://www.mintlify.com/docs/create/list-table#tables)을 확인해봤는데, Coinbase 측에서 따로 설정한 것은 없어보이고 mintlify 기본 디자인 자체가 예쁜 것 같다. mintlify의 Accordion의 경우에는 여러 Accordion 끼리 Group을 만들 수도 있다.

{{ figure(src="./img/vocsTableAndDetail.png", alt="vocsTableAndDetail", caption="vocs의 테이블과 디테일 섹션") }}

{{ figure(src="./img/mintlifyTableAndDetail.png", alt="mintlifyTableAndDetail", caption="mintlify의 테이블과 아코디언 섹션") }}

### API 명세 정의 기능 부재

OAS 등 REST API 명세 정의를 위한 기능이 따로 없다. 공식 레포의 Discussion의 [어느 한 유저의 글](https://github.com/wevm/vocs/discussions/27)에 따르면 [Scalar](https://scalar.com/)를 vocs에 얹어서 사용하는게 어떻겠냐고 제안했는데 검색 엔진도 2개고 UI 상으로 엄청 별로다.

vocs의 철학이나 본래 개발 목적상 벗어난 기능일 수도 있으나, OAS나 REST API 명세 정의를 위한 기능이 있었으면 좋겠다.

### Code Groups with Description, Nested Code Groups 기능 부재

여러 코드 블록을 탭으로 엮을 수 있는 Code Groups 기능이 있는데, vocs는 이 기능의 다양성이 부족하다.

가령, 코드 블록을 탭으로 엮으면서 코드 블록만 탭에 포함할 수 있지, 코드에 대한 부연 설명을 탭에 포함할 수는 없다. ([Coinbase Developer Platform | x402](https://docs.cdp.coinbase.com/x402/quickstart-for-buyers#1-install-dependencies)에서 `Node.js`와 `Python` 탭을 번갈아 누를 때마다 코드 블록 위에 있는 탭에 대한 설명도 바뀐다)

Code Group안에 또 다른 Code Group을 두어 탭 속 탭을 지원하지는 않는다. ([Coinbase Developer Platform | x402](https://docs.cdp.coinbase.com/x402/quickstart-for-buyers#3-make-paid-requests-automatically)에서 `Node.js`와 `Python` 탭을 번갈아 누를 때마다 하위에 있는 내용과 탭도 바뀐다)

사실 이 기능은 직접 컴포넌트를 작성하면 해결할 수 있다. 다만, UI 일관성을 위해서 코드 하이라이팅 등 기존 vocs에서 사용되던 것을 동일하게 적용해야 한다.

### 이미지 클릭시 확대 기능 부재

비교적 간단한 기능임에도 문서에 삽입된 이미지 클릭 시에 확대가 되는 기능이 없다. 이미지에 글자가 있으면 큰 글자가 아닌 이상 읽을 수가 없다. 공식 레포의 [Discussion](https://github.com/wevm/vocs/discussions/176)에 이 문제에 대해 언급되어 있는데, 아무런 대응이 없다.

## 결론

문서화 프레임워크나 SSG들이 결국에는 수렴 진화하게 되다보니 vocs가 다른 문서화 프레임워크에 비해 특출난 부분은 없는 것 같으면서도 Vite의 빠르고 가벼운 빌드에 만족할 수 있었다. 결국 문서화에 요구되는 여러 사항들을 잘 고려하여 적절한 문서화 프레임워크를 선택해야한다. 문서화에 요구되는 대표적인 사항들의 예와 이를 vocs에서 어떻게 처리하고 기능하는지에 아래에서 간략하게 정리하자면:

- **Docs As Code 철학을 행하기 위한 버전 관리와 같은 기능**: 온라인 에디터 기반이 아니라, `npm init vocs` 등으로 직접 설치하여 사용하는 문서화 프레임워크이기 때문에 완전히 Docs As Code 로 관리할 수 있다.
- **Customizable Component 지원, MDX**: vocs는 이용자가 직접 Vite를 통해 빌드하기 떄문에 컴포넌트를 완전히 자유롭게 구성할 수 있다.
- **문서 검색 엔진 커스텀화**: 기본적으로 [MiniSearch](https://lucaong.github.io/minisearch/types/MiniSearch.SearchOptions.html)를 사용하여 관련 옵션을 지원하고 Algolia 다른 검색 엔진으로 변경하는 것은 불가능한 것으로 보인다.
- **i18n과 같은 언어 지원**: 현재는 지원하지 않고, [관련 PR](https://github.com/wevm/vocs/pull/95)이 진행중인 것으로 보이나 머지가 확실하지 않다.
- **빌드 파이프라인**: Vite 기반, remark와 rehype 플러그인을 자유롭게 적용할 수 있다.
- **호스팅, 배포 용이성**: 가령, `npm run build`를 통해 빌드된 Static 파일들을 어떤 방법이던지 간에 배포하면 된다.
- **AI 관련 기능** (`llms.txt`, MCP Server): 빌드 시 `llms.txt`를 제공해준다. Vite 플러그인 기능으로 vocs에서는 25년 2월에 도입된 기능인데, 공식 문서에 이에 대한 언급은 없다. 다른 AI 관련 기능으로는 문서마다 _Ask in ChatGPT_ 나, _Copy page for LLMs_ 옵션이 있는 버튼을 선택해서 LLM 이용 용이성을 제공해주는 기능 정도가 있다.

vocs를 선택할 이유라면:

- **완전 무료**: 무료인데도 적당히 심미적이면서 기본적인 기능을 다 갖춤
- **빠르고 가벼움**: Vite로 빌드됨
- **높은 자유도**: 100% 자유롭게 개발 가능한 커스텀 컴포넌트, 자유로운 remark / rehype 플러그인 설정 가능
- **블록체인 관련 프로젝트일 경우**: 문서화 도구에 컬처핏을 굳이 따지자면 그나마 Crypto-Native? 내지는 Blockchain-Native한 문서화 도구임
- **TypeScript 관련 프로젝트일 경우 (Twoslash 지원)**: 사실 [rehype 플러그인](https://github.com/rehypejs/rehype-twoslash)으로 Twoslash 기능을 추가할 수 있기 때문에 플러그인 설정이 가능한 문서화 도구면 이 마저도 크게 매력적인 옵션이 아닐수도 있다. 실제로 [nextra](https://nextra.site/docs/advanced/twoslash)도 Twoslash 기능을 지원한다. 그래도 vocs가 좀 더 다양한 Twoslash Annotation에 따라 적절한 스타일이 반영된 코드 블록을 렌더 해주는 것 같다.

vocs의 두 특징, 컴포넌트를 자유롭게 구성할 수 있는 점과 블록체인 관련 프로젝트에서 사용하기 좋은 문서화 도구인 점을 고려한 아이디어가 있다. 좋은 아이디어인지는 모르겠지만, 지갑 연결이 필요한 가이드와 같은 페이지에서 문서와 메타마스크와 같은 지갑과 직접 연결하여 실습 과정을 제공하는 것이다. MDX에서 컴포넌트 작성이 자유롭다면 어떤 SSG든 사용가능한 아이디어긴하다. 공식 디스코드에서 vocs에서 wagmi의 React Hook을 어떻게 쓰면 좋을지 대화한 내역이 있어서 떠오른 아이디어다.

vocs를 선택하지 않을 이유라면:

- **유지 보수나 버그 대응이 느림**: 이슈나 PR 관리, 버그가 빠르게 대응되지 않음

사실 직접 겪은 대부분의 문제들이 스타일 관련 문제라서 그 부분은 컴포넌트를 커스텀해서 사용하면 해결된다. 문서를 심미적으로 만들기위해 컴포넌트를 작성하는 데 쓰는 노력과 비용도 중요하지만, 문서에서 글 자체가 잘 읽히도록 쓰는 노력과 비용도 중요하다고 생각해서 기본적으로 제공하는 컴포넌트도 심미적인 UI를 가졌으면 하는 아쉬움이 있다.

또, 이슈나 PR에 대한 대응이나 ToC가 제대로 표기되지 않는 큰 버그에 빨리 대응되지 않는 등 버그에 대한 대응도 느린 것도 상당히 아쉽다. 아무래도 MIT 라이선스 오픈소스고, wevm 팀이 vocs는 다른 viem이나 wagmi와 같은 TypeScript 툴을 위한 문서화 도구일 뿐이지 그들이 개발하는 메인 프로덕트는 아닌 것도 이유일 것이다. 우선순위를 따져보면, viem의 유지 보수에 투입할 자원을 문서화 도구인 vocs에 투입하지는 않을 것이다. 실제로 wevm 팀의 주요 개발자가 viem과 wagmi의 인기에 힘입어 vocs를 개발했다는 뉘앙스의 언급이 공식 레포의 [Discussion](https://github.com/wevm/vocs/discussions/74#discussioncomment-8377988)에 남아있다.

결론적으로 블록체인에 관한 내용을 명세해야한다면 사용하기에 나쁘진 않을 것 같다. 다만 이슈나 PR에 대한 대응이나 새로운 기능 개발과 기여가 활발해졌으면 좋겠다.
