---
title: "_Traffic4cast_ at NeurIPS 2021 – Temporal and Spatial Few-Shot Transfer Learning in Gridded Geo-Spatial Processes"
authors:
- Christian Eichenberger
- Moritz Neun
- Henry Martin
date: "2022-02-12T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2017-01-01T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
# publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: "PMLR"
publication_short: ""

abstract: The IARAI _Traffic4cast_ competitions at NeurIPS 2019 and 2020 showed that neural networks
can successfully predict future traffic conditions 1 hour into the future on simply aggregated
GPS probe data in time and space bins. We thus reinterpreted the challenge of forecasting
traffic conditions as a movie completion task. U-Nets proved to be the winning architecture,
demonstrating an ability to extract relevant features in this complex real-world geo-spatial
process. Building on the previous competitions, _Traffic4cast_ 2021 now focuses on the
question of model robustness and generalizability across time and space. Moving from one
city to an entirely different city, or moving from pre-COVID times to times after COVID
hit the world thus introduces a clear domain shift. We thus, for the first time, release data
featuring such domain shifts. The competition now covers ten cities over 2 years, providing
data compiled from over 10^12 GPS probe data. Winning solutions captured traffic dynamics
sufficiently well to even cope with these complex domain shifts. Surprisingly, this seemed to
require only the previous 1h traffic dynamic history and static road graph as input.

# Summary. An optional shortened abstract.
# summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

# tags:
# - Source Themes
# featured: false

# links:
# - name: ""
#   url: ""
url_pdf: https://proceedings.mlr.press/v176/eichenberger22a/eichenberger22a.pdf
# url_code: 'https://github.com/wowchemy/wowchemy-hugo-themes'
# url_dataset: ''
# url_poster: ''
# url_project: ''
# url_slides: ''
# url_source: ''
# url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: 'Image credit: [_Traffic4Cast2021_](https://www.iarai.ac.at/traffic4cast/)'
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""
---

<!-- {{% callout note %}}
Click the *Cite* button above to demo the feature to enable visitors to import publication metadata into their reference management software.
{{% /callout %}}

{{% callout note %}}
Create your slides in Markdown - click the *Slides* button to check out the example.
{{% /callout %}}

Supplementary notes can be added here, including [code, math, and images](https://wowchemy.com/docs/writing-markdown-latex/). -->
