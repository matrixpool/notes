test
==========================

.. tikz::
    \begin{tikzpicture}
    \draw (0,1) -- (1,0);
    \end{tikzpicture} 

.. tikz::
    \begin{tikzpicture}
    \draw (0pt,30pt) -- (30pt,0pt);
    \end{tikzpicture} 

.. tikz::
    \begin{tikzpicture}
    \draw (0,1) -- +(1,-1) -- +(1,1);
    \end{tikzpicture}

.. tikz::
    \begin{tikzpicture}
    \draw[->] (0,2.5) -- (3,2.5);
    \draw[<-] (0,2) -- (3,2);
    \draw[<->] (0,1.5) -- (3,1.5);
    \draw[|->] (0,1) -- (3,1);
    \draw[>->>] (0,0.5) -- (3,0.5);
    \draw[|<->|] (0,0) -- (3,0);
    \end{tikzpicture}

.. tikz::
    \begin{tikzpicture}[line width=3pt]
    \draw (0,0) -- (1,1) -- (2,0) -- cycle;
    \end{tikzpicture} 
