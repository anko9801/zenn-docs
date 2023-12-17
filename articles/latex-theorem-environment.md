

## 定理環境

```latex
\usepackage{amsthm}

% Define a new theorem environment
\newtheorem{theorem}{Theorem}
```

## 枠線

- ascmac
  - センテンス全体を枠線で囲むことができる
- fancybox
  - 行中に枠線で囲むことができる
- mdframed
  - より
- tcolorbox
- thmtools

```latex
\usepackage{ascmac}
\usepackage{fancybox}
\usepackage{mdframed}
\usepackage{tcolorbox}
```

```latex
% Customize the appearance of the theorem environment
\newtcolorbox{fancytheorem}{
  enhanced,
  colback=white,
  colframe=black,
  boxrule=0.5pt,
  arc=3mm,
  fonttitle=\bfseries,
  title=Theorem \thetheorem,
  attach title to upper,
  after title={\hspace{0.5em}},
  separator sign={\newline},
  description delimiters={\begin{center}}{\end{center}},
  before upper={\begin{description}},
  after upper={\end{description}},
}
```


\begin{fancytheorem}
  Let $a$ and $b$ be two real numbers. Then $a+b=b+a$.
\end{fancytheorem}

\usepackage{tcolorbox}
\usepackage{varwidth}
\tcbuselibrary{breakable}
\tcbuselibrary{skins}

\definecolor{frameinnercolor}{RGB}{49,44,44}

\newtcolorbox[auto counter,number within=section]{theobox}[1][]{%
enhanced,frame empty,interior empty,
coltitle=white,fonttitle=\bfseries,colbacktitle=frameinnercolor,
extras broken={frame empty,interior empty},
borderline={0.5mm}{0mm}{frameinnercolor},
sharp corners=downhill,
breakable=true,
top=4mm,
before skip=3.5mm,
attach boxed title to top left={yshift=-3mm,xshift=3mm},
boxed title style={boxrule=0pt,sharp corners=all},varwidth boxed title,
title=#1,
}

\newenvironment{theorem}[2][]{%
%#1 = タイトル,　#2 = 定理環境名
\ifstrempty{#1}{% ifstremptyはetoolbox.styで定義．etoolbox.styはtcolorboxが読み込むので宣言不要
\begin{theobox}[#2~\thetcbcounter.]}
{\begin{theobox}[#2~\thetcbcounter:~{#1}]}
}{\end{theobox}}
