# README.md
<h3>
Full name : TAN KOK HOW <br>
Email address : tkokhow@gmail.com
</h3>

<h3>Folder overview structure</h3>

- **ROOT**
    - src
        - regressor1.py
        - regressor2.py
        - classifier1.py
        - classifier2.py
    - README.md
    - eda.ipynb
    - requirements.txt
    - run.sh


<h3>How to execute the pipeline and modify any parameters</h3>

To execute pipeline, use "bash run.sh [model] [path]" 
Parameters: [model]: {regressor1 or regressor2 or classifer1 or classifer2}
                    where regressor1 is LinearRegression model (better of the two regressors)
                    where regressor2 is Lasso model (worse of the two regressors)
                    where classifer1 is KNeighborsClassfier model (better of the two classifers)
                    where classifer2 is SVC model (worse of the two classifers) 
            [path]: path to db that pipeline will be using to predict
Example: bash run.sh classifer1 data/new_popularity.db

<h3>Logical steps/flow of the pipeline</h3>

The pipeline proceeds through the following steps:

1. Connects to data/news_popularity.db
2. Query for combined data from all tables
3. Drop columns 'url' and 'weekday' as these are unneeded and interfere with the following step
4. One-hot encode combined data, effectively only encoding data_channel
5. Create new columns 
    * self_ref_shares_upp_var from self_reference_max_shares and self_reference_avg_shares
    * self_ref_shares_low_var from self_reference_min_shares and self_reference_avg_shares
    * kw_upp_var_max from kw_max_max and kw_avg_max
    * kw_low_var_max from kw_min_max and kw_avg_max
    * kw_upp_var_avg from kw_max_avg and kw_avg_avg
    * kw_low_var_avg from kw_min_avg and kw_avg_avg
6. Map shares variable into bins of logarithmic scale
7. Separate data into X and Y
8. Here, using a ColumnTransformer, columns:'ID','shares','timedelta','n_tokens_title','n_tokens_content','n_unique_tokens','n_non_stop_words','n_non_stop_unique_tokens','average_token_length','kw_min_min','kw_max_min','kw_avg_min','kw_min_max','kw_max_max' are removed as their correlation to shares are minimal.
9. N.A. values in the data are filled in with the mode of the dataset
10. Model then fits on the data and predicts a result.


<h3>Overview of key findings from the EDA conducted in Task 1 and the choices
made based on these findings</h3>

Dataset was acquired over a period of time. These dates were included as another variable, which is later removed due to effectiveness.<p>
Number of shares were more clustered below the average and sparse above the average. This, together with an understanding of the distrbution among articles made me choose a logarithmic scale for binning in the classification approach<p>
Shares are found to be not affected by time, therefore time elements are removed from the pipeline (including the date on which the data was acquired, mentioned above).<p>
There were a lot of features that the no. of shares were not well correlated with. This were removed, leaving the ML models lesser information to process, and be potentially faster without losing much accuracy<p>
As some max, avg and min statistics were given, upper range (max - avg) and lower range (avg - min) variables are created for these statistics.<p>
Surprisingly, shares are poorly correlated the number of words in titles and the content as well as the rate of unique or non-stop words in the content. In fact, shares are mainly correlated to n_comments.<p>
At the same time, some missing values are observed in the data, which will need to be filled it for the model to use.<p>
It is also seen that data_channel does not contain uppercase and so there are no duplicates of the same genre. Meanwhile, an one-hot encoder is used since the number of categories are very manageable.


<h3>Explanation of your choice of models</h3>

**Regression**

https://miro.medium.com/max/1674/1*_Wx0vKokbXd20HlbLKpj2A.jpeg

I went by the flowchart above, testing out viable models according to my earlier findings. 
I went through it like this:<p>
Checking the "Regression > Just Exploring" path...<br>
Checking the "Regression > Just Exploring > Poor Results?" path...<br>
Checking the "Regression > At least some knowledge > Non-linear > >10 > Yes > More Samples" path...<br>
Checking the "Regression > At least some knowledge > Linear > Yes" path...<br>
Checking the "Regression > At least some knowledge > Linear > No > Yes > Control/Interpretability > Yes" path...<p>
Of which, LinearRegression and Lasso Models did better based on minimising the root mean squared error. 
</p>    

**Classification**

https://scikit-learn.org/stable/_static/ml_map.png

I also went by the flowchart above, testing out viable models according to my earlier findings. 
I went through it like this:<p>
From the start of classification (at <100k samples):<br>
Checking the "Yes" path<br>
Checking the "Yes > Not Working" path<br>  
Checking the "Yes > Not Working > No" path<br>
Checking the "Yes > Not Working > No > Not Working" path<p>
Of which, KNeighborsClassifier and SVC Models did better based on minimising the root mean squared error.


<h3>Evaluation of the models developed</h3>

Of the models, based off the RMSE values calculated, regressors perform worse than classifiers. Both regressors show indication of being overtrained and exhibit high RMSE for the test set.
<p>
On the other hand, the classifers have a much lower RMSE for the test set, showing that it is able to predict better. Between both classifiers, SVC is overtrained with the RMSE of the train set being much lower than the RMSE of the test set, while KNeighborsClassifier is alright. KNeighborsClassifier also exhibits the lowest RMSE of all models, showing that it is able to best predict the shares of each article.
    

<h3>Other considerations for deploying the models developed</h3>

The classifiers and regressors should be fast in doing their prediction. SVC is slow, but it is still the second best classifier out of those that are tried.
