---
layout: post
title:  "Handle the untracked"
date:   2017-07-10 14:42:29
categories: Git
excerpt: Git, Github, .gitignore, untracked
---

Git으로 프로젝트를 관리하다가 새로 추가한 directory나 file이 추적되지 않을 때가 있다.
즉, git status 명령을 통해 새로 추가된 것이 표시되어야 하는데 그렇지 않은 경우다.
<br>
이런 문제가 발생하면 기본적으로 프로젝트의 .git파일이 있는 폴더에 위치한 .gitignore를 체크한다.
>.gitignore
><pre><code>direct
.js
image.png
</code></pre>
위와 같은 경우에 Git은 direct 폴더와 js 확장자의 파일들, image.png 파일을 무시하고 있다.

<br>
.gitignore에 있지 않음에도 불구하고 문제가 지속되면 directory와 file 두 경우로 나누어 볼 수 있다.

<br>
* 빈 directory는 인식하지 않을 수 있다.
이 경우엔 해당 directory에 아무 파일이나 생성하면 되는데, 빈 directory가 필요한 경우 .gitignore나 .keep 파일을 생성해주면 된다.
><pre><code>touch .keep
</code></pre>

<br>
* Git이 관리하는 프로젝트의 상위폴더도 검사한다.
과거에 프로젝트를 local disk에 생성하였거나 그 밖의 이유로 상위 폴더 어딘가에 또 다른 .gitignore 또는 .gitignore_global이 생성되어 있으면 하위 폴더의 Git 프로젝트에도 영향을 미친다. 해당 파일을 찾아 지워주거나 수정해준다.
