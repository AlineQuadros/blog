---
layout: post
title:  "Quick script for GCMS data and metabolomics"
author: aline
categories: [ data-science, ecology ]
image: assets/images/gcms.jpg

---

<style type="text/css">
.main-container {
  max-width: 940px;
  margin-left: auto;
  margin-right: auto;
}
code {
  color: inherit;
  background-color: rgba(0, 0, 0, 0.04);
}
img {
  max-width:100%;
  height: auto;
}
.tabbed-pane {
  padding-top: 12px;
}
button.code-folding-btn:focus {
  outline: none;
}
</style>



<div class="container-fluid main-container">

<!-- tabsets -->
<script>
$(document).ready(function () {
  window.buildTabsets("TOC");
});
</script>

<!-- code folding -->






<div class="fluid-row" id="header">



<h1 class="title toc-ignore">clustering of GC-MS peaks</h1>
<h4 class="author"><em>script by Aline Quadros</em></h4>
<h4 class="date"><em>16 February 2017</em></h4>

</div>


<div id="import-original-excel-file" class="section level2">
<h2>import original Excel file</h2>
<pre class="r"><code>library(xlsx)</code></pre>
<pre><code>## Warning: package 'xlsx' was built under R version 3.3.2</code></pre>
<pre><code>## Loading required package: rJava</code></pre>
<pre><code>## Loading required package: xlsxjars</code></pre>
<pre><code>## Warning: package 'xlsxjars' was built under R version 3.3.2</code></pre>
<pre class="r"><code>setwd(&quot;Z:/__script GCMS/&quot;)

run_time &lt;- 60  # set in

input_t &lt;- read.xlsx(&quot;input.xlsx&quot;, 1 )
colnames(input_t) &lt;- c(&quot;original_peak&quot;, &quot;rt&quot;, &quot;base_peak&quot;, &quot;mz&quot;, &quot;area&quot;, &quot;sample_name&quot;)
input_t$sample_name &lt;- as.character(input_t$sample_name)
input_t$sample_name &lt;- substr(input_t$sample_name, 45, 50)
input_t$sample_name &lt;- as.factor(input_t$sample_name)

input_t$rt &lt;- as.character(round(input_t$rt, digits = 2))

str(input_t)</code></pre>
<pre><code>## 'data.frame':    2025 obs. of  6 variables:
##  $ original_peak: num  1 2 3 4 5 6 7 8 9 10 ...
##  $ rt           : chr  &quot;6.16&quot; &quot;6.25&quot; &quot;6.41&quot; &quot;6.53&quot; ...
##  $ base_peak    : num  105 110 117 96.1 103 115 108 97.1 107 91 ...
##  $ mz           : num  105 110 117 96.1 103 115 108 97.1 107 91 ...
##  $ area         : num  174324 2568353 256358 980743 425833 ...
##  $ sample_name  : Factor w/ 27 levels &quot;-_176.&quot;,&quot;_125.D&quot;,..: 1 1 1 1 1 1 1 1 1 1 ...</code></pre>
</div>
<div id="create-dataframe-that-will-sort-per-retention-time" class="section level2">
<h2>create dataframe that will sort per retention time</h2>
<pre class="r"><code>dataset_rt &lt;- data.frame(sequential_rt=1:6000, stringsAsFactors = FALSE)
seqs &lt;- seq(0.01, run_time, by=0.01)
dataset_rt$sequential_rt &lt;- as.character(seqs, digits = 2)</code></pre>
</div>
<div id="organize-datasets" class="section level2">
<h2>organize datasets</h2>
<pre class="r"><code>options(digits = 3)
for (l in 1:nlevels(input_t$sample_name)) {
  sub &lt;- input_t[input_t$sample_name == levels(input_t$sample_name)[l], c(1,2, 4)]
  #create two columns to receive ID and MASS of this sample
  colu &lt;- ncol(dataset_rt)
  dataset_rt[, colu+1] &lt;- 0
  dataset_rt[, colu+2] &lt;- 0
  colnames(dataset_rt)[colu+1] &lt;- paste(as.character(levels(input_t$sample_name)[l]), &quot;__ID&quot;)
  colnames(dataset_rt)[colu+2] &lt;- paste(as.character(levels(input_t$sample_name)[l]), &quot;__mz&quot;)
  for (lin in 2 : nrow(sub)) {  ##tem erro verificar
    lin2 &lt;- match(sub[lin, &quot;rt&quot;], dataset_rt$sequential_rt)
    dataset_rt[lin2, colu+1] &lt;- sub[lin, &quot;original_peak&quot;] #ID = peak number
    dataset_rt[lin2, colu+2] &lt;-  as.double(sub[lin, &quot;mz&quot;], digits = 1)  # mass
  }
}
dataset_rt_red&lt;-data.frame(stringsAsFactors = FALSE)
co&lt;-0
nc &lt;- ncol(dataset_rt)
#trim  lines with only zeros
for (m in 1:nrow(dataset_rt)){
  if (sum(dataset_rt[m, 2:nc]) != 0)  {
   co &lt;- co + 1
   dataset_rt_red &lt;- rbind(dataset_rt_red, dataset_rt[m,])
  }
}

dataset_rt_red &lt;- dataset_rt_red[, seq(from = 3, to = nc, by=2) ]</code></pre>
</div>
<div id="set-buffer" class="section level2">
<h2>Set buffer</h2>
<pre class="r"><code>buf &lt;- 5 # variation in retention time considered
nc2 &lt;- ncol(dataset_rt_red)</code></pre>
</div>
<div id="create-dataset-of-all-compounds-taking-buffer-into-account" class="section level2">
<h2>create dataset of all compounds taking buffer into account</h2>
<pre class="r"><code>dataset_list &lt;- data.frame(stringsAsFactors = FALSE)
dataset_list_ord &lt;- data.frame(stringsAsFactors = FALSE)

for (c in 1:nc2){
  for (l in 1:nrow(dataset_rt_red)) {
      if (dataset_rt_red[l,c]!= 0) {
        newrec &lt;- c(rownames(dataset_rt_red)[l], dataset_rt_red[l,c])
        dataset_list &lt;- rbind(dataset_list, as.numeric(newrec))
      }
  }
}
colnames(dataset_list) &lt;- c(&quot;ret_time&quot;, &quot;mass&quot;)
dataset_list_ord &lt;- dataset_list[order(dataset_list$mass, dataset_list$ret_time),]

#eliminate exact duplicates
dataset_list_ord &lt;- dataset_list_ord[!duplicated(dataset_list_ord), ]

#merge according to buffer size
dataset_compounds &lt;- data.frame(stringsAsFactors = FALSE)

cind &lt;- 0
p&lt;-1

while (p &lt;= nrow(dataset_list_ord)){
  # add coumpound
  cind &lt;- cind + 1

  dataset_compounds[cind, &quot;name&quot;] &lt;- paste(&quot;compound_&quot;, cind, sep=&quot;&quot;)
  dataset_compounds[cind, &quot;min_ret_time&quot;] &lt;-  dataset_list_ord[p, &quot;ret_time&quot;]
  dataset_compounds[cind, &quot;max_ret_time&quot;]  &lt;- dataset_list_ord[p, &quot;ret_time&quot;]
  dataset_compounds[cind, &quot;mass&quot;] &lt;- dataset_list_ord[p, &quot;mass&quot;]

  # check next rows according to buffer to see if theres more
  n_added &lt;- 0
  for (b in 1:buf-1) {
    if (dataset_list_ord[p, &quot;mass&quot;] == dataset_list_ord[p+b, &quot;mass&quot;]){
      #next line is same compund - update ret time
      dif &lt;- dataset_list_ord[p+b, &quot;ret_time&quot;] - dataset_compounds[cind, &quot;min_ret_time&quot;]
      if (dif &lt;=5 &amp; dif &gt;=0){
          dataset_compounds[cind, &quot;max_ret_time&quot;] &lt;- dataset_list_ord[p+b, &quot;ret_time&quot;]
          n_added &lt;- n_added+1   
      }

    } else {

      break
    }
  }
  p &lt;- p+1+n_added
}</code></pre>
<div id="finally-create-matrix-with-presenceabsence-of-each-compound-per-sample" class="section level3">
<h3>finally, create matrix with presence/absence of each compound per sample</h3>
<pre class="r"><code>dataset_final &lt;- matrix(nrow=ncol(dataset_rt_red), ncol=nrow(dataset_compounds)+1)
dataset_final &lt;- as.data.frame(dataset_final)
colnames(dataset_final)[1] &lt;- &quot;sample&quot;
colnames(dataset_final)[2:nrow(dataset_compounds)+1] &lt;- dataset_compounds$name</code></pre>
<pre><code>## Warning in colnames(dataset_final)[2:nrow(dataset_compounds) + 1] &lt;-
## dataset_compounds$name: number of items to replace is not a multiple of
## replacement length</code></pre>
<pre class="r"><code>i &lt;-1
for (cc in 1:ncol(dataset_rt_red)){
  # read each sample
  dataset_final[i, &quot;sample&quot;] &lt;- colnames(dataset_rt_red)[cc]
  for (nr in 1:nrow(dataset_rt_red))
    if (dataset_rt_red[nr,cc] != 0){
      ma &lt;- dataset_rt_red[nr,cc]
      rt &lt;- as.numeric(rownames(dataset_rt_red)[nr])
      com_id &lt;- dataset_compounds[dataset_compounds$mass == ma
                                  &amp; dataset_compounds$min_ret_time &lt;= rt
                                  &amp; dataset_compounds$max_ret_time &gt;= rt , &quot;name&quot;]
      dataset_final[i, com_id] &lt;- 1
    }
  i&lt;-i+1
}

dataset_final[is.na(dataset_final[,]) ] &lt;- 0

write.csv(dataset_final, &quot;out.txt&quot;)</code></pre>
</div>
</div>
<div id="multivariate-analyses" class="section level2">
<h2>Multivariate analyses</h2>
<pre class="r"><code>#pca
datt &lt;- dataset_final[, 3:ncol(dataset_final)]
rownames(datt) &lt;- dataset_final$sample
simi &lt;- hclust(dist(datt))
plot(simi)</code></pre>

</div>




</div>

<script>

// add bootstrap table styles to pandoc tables
function bootstrapStylePandocTables() {
  $('tr.header').parent('thead').parent('table').addClass('table table-condensed');
}
$(document).ready(function () {
  bootstrapStylePandocTables();
});


</script>

<!-- dynamically load mathjax for compatibility with self-contained -->
<script>
  (function () {
    var script = document.createElement("script");
    script.type = "text/javascript";
    script.src  = "https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML";
    document.getElementsByTagName("head")[0].appendChild(script);
  })();
</script>
