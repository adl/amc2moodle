# `amc2moodle`

`amc2moodle`, is **a tool to convert automultiplechoice quizz (v1.0.3-v1.4) to moodle questions (XML format).**
The automuliplechoice LaTeX format is convenient, and can be used for preparing question and avoiding moodle web gui.

## How it works
The first conversion step is to convert the LaTeX file into XML file. This is performed by [LaTeXML](https://dlmf.nist.gov/LaTeXML/) and allow user defined command.
Then, other transformation are applied in python with XSLT stylesheet. Most of LaTeX possibilities are supported (equations, tables, graphics, user defined commands).
The question can then be imported in the moodle question bank using category tags.

> Examples of supported AMC question are provided in the [test](./test/) folder.

## Installation

### before installing amc2moodle :

  -  install python (version >=3.5)
  -  install`imageMagick`, useful to convert image files (*.eps, *.pdf, ...) into png
      - ubuntu : `sudo apt-get install imagemagick`
  -  install [`LaTeXML`](http://dlmf.nist.gov/LaTeXML) [tested with version 0.8.1] This program does the first step of the conversion into XML
      - ubuntu : `sudo apt-get install latexml`
      - see also [LaTeXML wiki](https://github.com/brucemiller/LaTeXML/wiki/Installation-Guides) or [install notes](https://dlmf.nist.gov/LaTeXML/get.html) that all the dependencies are installed (perl, latex, imagemagick).
  -  install `xmlindent` [optional]. This program can be used to indent well the XML file
      - ubuntu : `sudo apt-get install xmlindent`

For Macos users, most dependencies can be installed with `brew` but `LaTeXML` installation can failed for some version. Please see the steps given in the install script [workflow](.github/workflows).


### Install with pip

Run
```
pip install .
``` 
in the root folder (where `setup.py` is). This will automatically install other dependencies ie `lxml`, `Wand`.
Alternatively, you can run
```
pip install . -e
``` 
to install it in editable mode, usefull if git is used.



Note : for ubuntu users use `pip3` instead of pip for python3.

### Uninstallation
Run `pip uninstall amc2moodle`.

## Convertion
The program can be run in a shell terminal,
```
amc2moodle input_Tex_file.tex -o output_file.xml -c catname
```
Help can be obtained using
```
amc2moodle -h
```

Then on moodle, go to the course `administration\question bank\import` and choose 'moodle XML format' and tick : **If your grade are not conform to that you must use : 'Nearest grade if not listed' in import option in the moodle question bank** (see below for details).


## Usage

### What you can do
Examples of the `amc2moodle` possibilities are given at [QCM.pdf](./test/QCM.pdf)

  -  Convert `question` and `questionmult` environments.
  -  You don't need to remove questionnaires part `\exemplaire` or `\onecopy`. But if this part contains undefined commands, remove/comment it!
  -  Put in-line equations like $x^2$ or use equation environment (or $$ delimiters). For the moment eqnarray  or the amsmath environments multline, align are not supported. The choice have been made to keep equation in tex and use mathjax filter of moodle for rendering. In my opinion, it is better for modifying question after importation.
  -  Include image, in all format supported by `Wand`. `amc2moodle`  will convert it in .png for moodle export. The image will be embedded as text (base64) in the output xml file. The folder is '/' in moodle. The image can be in an another folder than the tex file.
  -  Include Table, with the tabular environment. In the present form, `amc2moodle` put  border around each cell.
  -  Use italic, typerwritter, bold, emphasize...
  -  Automatically add an answer like ``there is no good answer'' if there is no good answer.
  -  Use user's command defined in the LaTeX file.
  -  Use `\usepackage[utf8]{inputenc}`   for accents
  -  Use packages that are supported by `LaTeXML`. See the list [here](http://dlmf.nist.gov/LaTeXML/manual/included.bindings). Instead you need to add a binding to LaTeXML.
  -  Use `tikz`. `LaTeXML` generates `svg` content, embedded in the question or answer html text.
  -  All answers are Shuffled by default, you can keep the answer initial order by setting `ShuffleAll = False` in `grading.py`


### What you cannot do
  -  Use underscore in question name field !
  -  Use verbatim. This environment is not supported by `automultiplechoice` 1.0.3. Use `alltt` package instead.
  -  Use font size (easy to add)
  -  Use amsmath environments like align, aligned... Because  `text` attribute of `\elem{equation}`, provided by `LaTeXML` output, doesn't contains really the raw tex equation.
  -  Change border of table
  -  Use command like `\raggedright`, text align is not fully supported. This add align information into the `class` attribute of `\elem{note}` and the string matching break down. Note that `\raggedright` is bypassed.
  -  Usage of `multicol` is bypassed. But it should be possible to use it elsewhere (create newcommand).
  -  Translate equation into mathml, but it can be easily changed
  -  Use AMC numeric, or open question
  -  Only the main commands of the package `automultiplechoice.sty` are supported in french. The english keywords support is on-going. The list of supported keywords can be seen in `automultiplechoice.sty.ltxml`
  -  You cannot remove the add of "None of these answers are correct" choice at the end of each multiple question.

 
### Grading strategy
In moodle 3, the grading strategy is different from AMC, especially, for questions with multiple answers. In this case, AMC affects a grade for each checked good answer and each non-checked wrong answer. The total grade of the question depend on the number of choice.

In Moodle, only checked item leads to a grade, positive or negative. The grading is compute in the xxx.py script.
The defaut grading parameter are set in xxx.py script to
```
# Multiple :: e :incohérence, b: bonne,  m: mauvaise,  p: planché
amc_bs = {'e':-1,'b':1,'m':-0.5}              
amc_bm = {'e':-1,'b':1,'m':-0.5, 'p':-1}   
# defaut question grade in moodle
moo_defautgrade = 1.                       
```
This value can be changed (as in AMC) with the tex command
```
\baremeDefautS{e=-0.5,b=1,m=-0.5}         % never put b<1,
\baremeDefautM{e=-0.5,b=1,m=-0.25,p=-0.5} % never put b<1,
```
or at the question level with the tex command `\bareme`.
The gade $g_i$ in % is then computed as
$g_i = 100  c_i / N_i$ where $i$ stand for the good or the wrong answer. Here, $N_i$ is the total number of the good or the wrong answer and $c_i$ the coefficient (m, b, ...). It important to set b=1 to get 100% if all the good answers are found. The e parameter is not used here, because it is not possible to tick 2 answers in moodle for one-answer-question. The only case where incoherent can be used is if the ``_there isn't any correct answer_'' answer is ticked with another question but it is not implemented.
For instance if `m=-0.5` and `b=1`, a student who ticks all the wrong answers get -0.5, a student who ticks all the good answer get  1 and student who ticks all the boxes get 0.5.

Another difference is that moodle 3 use tabulated grade like : 1/2, 1/3, 1/4, 1/5, 1/6, 1/7, 1/8, 1/9, 1/10 and their multiple. **If your grade are not conform to that you must use : 'Nearest grade if not listed' in import option in the moodle question bank**. But check at least that the sum of good answer give 100% !



### Categories
By default, the imported questions are all created in `$course$/filein`. When the category flag is used, the AMC command `element` is used to create subcategories and the argument `catname` is used instead of `filein`.
Each question is then placed in `$course$/catname/elementName`.


### Feedback
Feedback are present, in a certain way, in `automuliplechoice` with the `\explain` command. This part is not yet implemented here. However it could be easy to add it at the response or question level as other fields and bypass them for real `automuliplechoice` test.


## Troubleshooting
In case of problem, do not hesitate to ask help on  [issues](https://github.com/nennigb/amc2moodle/issues)
  - 'convert: not authorized..' see ImageMagick policy.xml file

## Related Project
  - [auto-multiple-choice](https://www.auto-multiple-choice.net),  is a piece of software that can help you creating and managing multiple choice questionnaires (MCQ), with automated marking.
  - [TeX2Quiz](https://github.com/hig3/tex2quiz), is a similar project to translate multiple choice quiz into moodle XML, without connexion with AMC.
  - [moodle](https://www.ctan.org/pkg/moodle) - Generating Moodle quizzes via LaTeX. A package for writing Moodle quizzes in LaTeX. In addition to typesetting the quizzes for proofreading, the package compiles an XML file to be uploaded to a Moodle server.

## How to contribute ?
If you want to contribute to `amc2moodle`, your are welcomed! Don't hesitate to
  - report bugs, installation problems or ask questions on [issues](https://github.com/nennigb/amc2moodle/issues)
  - propose some enhancements in the code or in documentation through **pull requests** (PR)
  - create a moodle plugin for import
  - support new kind of questions
  - add support for other language (french and english are present)
  - ...

To ensure code homogeneity among contributors, we use a source-code analyzer (eg. pylint).
Before submitting a PR, run the tests suite.

## License
This file is part of amc2moodle, a tool to convert automultiplechoice quizz to moodle questions.
amc2moodle is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
amc2moodle is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.
You should have received a copy of the GNU General Public License along with amc2moodle.  If not, see <https://www.gnu.org/licenses/>.
