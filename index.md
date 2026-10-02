---
layout: default
title: Zillow Prize
description: A gradient boosted machine for Zillow's log error between the Zestimate and the sale price.
samwiki: true
---

<div class="sw-lede">
  <div class="sw-lede-copy">
    <h2>What the script fits</h2>
    <p>This repository is the working script for the <a href="https://www.kaggle.com/c/zillow-prize-1">Zillow Prize</a>, the Kaggle contest that asked for the error in Zillow's own home-value estimate. If <code>Z</code> is the Zestimate and <code>S</code> is the sale price, the target is <code>logerror = log(Z) - log(S) = log(Z / S)</code>. A prediction of 0 says the Zestimate already matches the sale. The submission is one row per parcel and six columns: October, November, and December of 2016, then the same three months in 2017. The contest discussion lives at <a href="https://www.kaggle.com/c/zillow-prize-1/discussion">the Zillow Prize forum</a>.</p>
    <p>Sales in <code>train_2016_v2.csv</code> are left-joined to <code>properties_2016.csv</code> on <code>parcelid</code>, so every training sale keeps its property attributes. Identifier fields that arrive as numbers are recoded as factors: building quality type, FIPS, heating system, property land use, census tract and block, city, county, and unit count. The sale month is taken from <code>transactiondate</code> with <code>lubridate::month</code> and stored as a factor. The date column is then dropped. The script's own note is that month of the year has predictive power, which is why the six contest months are not one shared forecast.</p>
    <p><code>make_features</code> builds the columns that sit on top of the raw assessor file. <code>N_value_ratio</code> is <code>taxvaluedollarcnt / taxamount</code>, assessed value per dollar of tax. <code>N_living_area_prop</code> is <code>calculatedfinishedsquarefeet / lotsizesquarefeet</code>. <code>N_tax_score</code> is the product <code>taxvaluedollarcnt * taxamount</code>. <code>N_zip_count</code> is the number of parcels sharing <code>regionidzip</code>. A structure-tax deviation, <code>abs(structuretaxvaluedollarcnt - N_Avg_structuretaxvalue) / N_Avg_structuretaxvalue</code>, is still written in the function. The <code>group_by(regionidcity)</code> that would create <code>N_Avg_structuretaxvalue</code> is commented out, so that deviation is not an input the current script can compute. Raw census tract, zoning description, census tract and block, and assessment year are dropped inside the same function.</p>
    <p>Columns with more than 80 percent missing values are removed from the training frame, and the same names are removed from the property file used at score time. Training rows are kept when <code>logerror</code> is between <code>-0.39</code> and <code>0.4</code>. On the multiplicative scale those bounds are <code>exp(-0.39) ≈ 0.677</code> and <code>exp(0.4) ≈ 1.492</code>, so the fit sees sales where the Zestimate runs from about 0.68 times the sale price to about 1.49 times the sale price. The model is <code>gbm::gbm</code> with <code>distribution = "gaussian"</code>, squared error on <code>logerror</code>, using every remaining column. The source script sets 600 trees, <code>interaction.depth = 5</code>, <code>shrinkage = 0.0033</code>, and <code>bag.fraction = 0.8</code>, and it uses half of <code>detectCores()</code>. Depth 1 would be an additive model. Depth 5 allows a tree whose splits combine up to five variables. <code>xgboost</code> is loaded with the other packages. The call that is fit is <code>gbm</code>.</p>
    <p>Scoring builds a month proxy so the property file has the same factor the training rows used. <code>predict.gbm</code> is called with all 600 trees after the month is set to <code>"10"</code>, <code>"11"</code>, and <code>"12"</code>, and those three vectors are written as <code>201610</code>, <code>201611</code>, and <code>201612</code>. The 2017 columns <code>201710</code>, <code>201711</code>, and <code>201712</code> are filled with 0, the forecast that the log residual is zero a year later. The script writes <code>submission_with_new_features.csv</code> with <code>scipen = 999</code> so the parcel ids stay in decimal form. That CSV is not in the repository, and neither is a public leaderboard score. The knitted notebook is an earlier pass of the same file: 200 trees, <code>shrinkage = 0.033</code>, and the knit stops with <code>transactiondate</code> not found and then an invalid <code>-drop.column</code>, before a model is saved.</p>
  </div>
  <aside class="sw-find" aria-label="Model facts">
    <h2>Model facts</h2>
    <ul>
      <li><strong>Target</strong> <code>logerror = log(Zestimate) - log(SalePrice)</code>. A fitted value of 0 leaves the Zestimate unchanged.</li>
      <li><strong>Join</strong> <code>train_2016_v2.csv</code> left-joined to <code>properties_2016.csv</code> on <code>parcelid</code>.</li>
      <li><strong>Engineered columns</strong> Tax value over tax amount, finished area over lot size, the product of tax value and tax amount, parcel count in the zip, and sale month.</li>
      <li><strong>Rows and columns kept</strong> <code>logerror</code> from -0.39 to 0.4, about 0.68 to 1.49 times the sale price. Columns above 80 percent missing are dropped.</li>
      <li><strong>Gaussian GBM</strong> 600 trees, interaction depth 5, shrinkage 0.0033, bag fraction 0.8. The knitted notebook is the earlier 200-tree, shrinkage 0.033 run, and it errors before a fit.</li>
      <li><strong>What gets submitted</strong> October–December 2016 from <code>predict.gbm</code>. The three 2017 months are 0.</li>
    </ul>
  </aside>
</div>

<section class="sw-section" id="notebook">
  <div class="sw-section-head">
    <h2>Notebook <span class="sw-pill">R</span></h2>
    <p>The source script is the 600-tree model. The HTML file is the earlier knit, errors included.</p>
  </div>
  <div class="sw-grid">
    <article class="sw-card">
      <div class="sw-card-top">
        <h3 class="sw-name">Gradient boosted machine</h3>
        <span class="sw-lang">R Markdown</span>
      </div>
      <p class="sw-desc">Join, factor recodes, the four engineered columns, the 80 percent missingness screen, the log-error filter, and the 600-tree Gaussian GBM.</p>
      <p class="sw-meta">zillow_gbm.Rmd</p>
      <div class="sw-actions">
        <a class="sw-btn sw-btn-source" href="https://github.com/sdcastillo/Kaggle-Zillow-Competition/blob/master/zillow_gbm.Rmd">View source</a>
      </div>
    </article>
    <article class="sw-card">
      <div class="sw-card-top">
        <h3 class="sw-name">Knitted notebook</h3>
        <span class="sw-lang">HTML</span>
      </div>
      <p class="sw-badge">Earlier run</p>
      <p class="sw-desc">200 trees and shrinkage 0.033. The knit reports <code>transactiondate</code> not found, then fails on <code>drop.column</code>, and does not save a model.</p>
      <p class="sw-meta">zillow_gbm.nb.html</p>
      <div class="sw-actions">
        <a class="sw-btn sw-btn-live" href="zillow_gbm.nb.html">Open notebook</a>
      </div>
    </article>
  </div>
</section>
