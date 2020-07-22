# Air pollution prediction using Recurrent Neural Networks (with an Attention mechanism)

Air pollution levels in North Macedonia are 20 times over the limit imposed by EU. As a result, it is estimated that it contributes for premature deaths in Skopje, Tetovo and Bitola. The ability to predict the air pollution levels would prove to be of significant help in improving this unwanted statistic. The idea is to examine the trends in historical data generated by the 5 government air pollution sensors in Skopje, with the hope of gaining a better understanding of the situation we are currently in. In this research project we compared the performances of two main architectures of LSTM Recurrent Neural Network: standard seq2seq models and seq2seq models with an attention mechanism. The models with an attention mechanism proved to be superior by outperforming by all metrics on every dataset.

## 1. Attention
In this section we briefly discuss the advantages of the attention mechanism and refer the readers to [Bahdanau et al. “Neural Machine Translation by Jointly Learning to Align and Translate.” CoRR abs/1409.0473 (2015): n. pag.] for further details.    
Standard encoder-decoder RNN models work in such a way that the encoder module processes the input sequence, produces a hidden (thought) vector encapsulating the necessary information from the input and passing it to the decoder module in order to decode the output sequence. A potential issue with this approach is that the model is forced to compress all the necessary information into a fixed-length vector, which may be difficult to achieve given long sequences.  
In order to address this issue, the authors introduce an extension to the encoder-decoder model which learns to align and translate jointly. Each time the proposed model decodes an element of the output sequence, it searches for a set of positions in the input sequence where the most relevant information is concentrated. The model then predicts a target vector based on the context vectors associated with these source positions and all the previously generated elements of the output sequence.  
The most important distinguishing feature of this approach from the standard encoder-decoder is that it does not attempt to encode the whole input sequence into a single fixed-length vector. Instead, it encodes the input sequence into a sequence of vectors and chooses a subset of these vectors adaptively while decoding the output sequence. This frees the model from having to squash all the information into a fixed-length vector. 
[image of basic attention]

## 2. Related work
In this section we briefly summarize related work for forecasting air pollution. In [Buiet al. (2018). A Deep Learning Approach for Forecasting Air Pollution in South Korea Using LSTM; arXiv:1804.07891] the authors use standard LSTM seq2seq models; In [Fei et al. (2019). A Model-driven and Data-driven Fusion Framework for Accurate Air Quality Prediction] the authors use an LSTM model as a corrector for predictions made by the standard CMAQ method; In [Divyam et al. (2019). VayuAnukulani: Adaptive Memory Networks for Air Pollution Forecasting] the authors use seq2seq models with an attention mechanism, but without teacher forcing or decoder input; In [Grégoire et al. (2020). DeepPlume: Very High Resolution Real-Time Air Quality Mapping] the authors discuss a system for air pollution prediction on a large scale. Regarding research outside of this domain, [Sangyeon et al. (2019). Financial series prediction using Attention LSTM; arXiv:1902.10877] and [Qiu J et al. (2020) Forecasting stock prices with long-short term memory neural networks based on attention mechanism. PLoS ONE 15(1): e0227222] demonstrate the superiority of attentive seq2seq models in the field of finance, while [Gangjun et al. (2019). Research on Short-Term Load Prediction Based on Seq2seq Model. Energies. 12. 3199. 10.3390/en12163199] in the field of electricity load prediction; The authors in [Fazil et al. “Deep Learning for Renewable Power Forecasting: An Approach Using LSTM Neural Networks.” International Journal of Energy and Power Engineering 12 (2018): 416-420] experiment with standard seq2seq models for renewable power forecasting.

## 3. Datasets
In total, three datasets were used for the task of air pollution prediction:
1. Air pollution data generated by 5 government sensors in possesion of the Ministry of Environment and Physical Planning of North Macedonia (https://www.moepp.gov.mk/), placed in several locations in Skopje, North Macedonia: Centar, Karpos, Lisice, Rektorat and Miladinovci. Each of the sensors measure a different subset of pollutants, with PM<sub>10</sub> being measured by all of them, hence why this was set as the target feature we are predicting. 
2. Weather data fetched from Dark Sky (https://darksky.net/) for each of the sensor locations
3. Generated datetime features (hour of day, day of week, month of year, weekend flag, holidayflag). 

Each of the datasets were on hourly basis, with Timestamp being the index column. 

## 4. Data analysis
In this and the following section we will be exploring the dataset for the station of Centar, however the same exploratory analysis was performed on all datasets.
From Figure 11 we can easily notice the seasonality of the pollution levels.  
Upon observing Figure 12, we can confirm that the air pollution is highest during the winter months and progressively decreases as summer months approach.  
From Figure 14 we notice that when we first stratify the data by season of year, then by day of week, the marginal difference of the pollution between the days is higher during the winter period than it is during the summer.  
Furthermore, from Figure 15 we notice that the marginal difference of pollution between the periods of the day is most notable during the winter months, with the periods between 9pm and 3am being the most polluted.

## 5. Data preprocessing
### 5.1. Air pollution datasets
As we mentioned earlier, each sensor measures only a subset of the pollutants. Upon observing the plot for missing values for the air pollution datasets (Figure 17), we decided to discard all of the pollutants except with PM<sub>10</sub> and with PM<sub>2.5</sub>. Furthermore, the dataset was truncated and we only used measurements from 2011 onward. 
From the histogram of sensor inoperability duration (Figure 18), we notice that the periods are usually short and last only a few hours. 
Naturally, PM measurements have exponential distribution (Figure 19, left) and have a few outliers. Usually, machine learning algorithms converge faster when the data is scaled. If we decide to normalize this feature in a certain range, due to its distribution and outliers, the values will generally fall in the lower section of the range. On the other hand, if we want to standardize the feature, it needs to have Gaussian distribution beforehand. If we log-transform the feature, not only do we transform its distribution to Gaussian (Figure 19, right), but we suppress the outliers as well, solving both problems at once. This transfromation is performed both on PM<sub>10</sub> and PM<sub>2.5</sub> and then, standardize these features with mean 0 and std 1. Similar transformations were performed on the other air pollution datasets. 
### 5.2. Weather datasets
Upon observing the plot for missing values for the weather datasets (Figure 17), we decided to discard the following features: icon, precipAccumulation, precipAccumulation.1, precipType, ozone, pressure, summary and windGust. Accordingly, we used the following weather features: temperature, apparentTemperature , cloudCover, dewPoint, humidity, precipIntensity, precipProbability, uvIndex, visibility, windBearing and windSpeed. Moreover, due to the cycling nature of the wind bearing feature, it was projected into two separate ones with the sin/cos transformation in order to capture the spatial proximity.  
The temperature, humidity and dewpoint naturally have Gaussian distribution, so these features were standardized with mean 0 and std 1, while the others were normalized in the range [0, 1].  
### 5.3. Datetime datasets
This dataset doesn't have missing values since it is generated. As before, due to the cyclic nature of the hour, day and month, these features were projected into two separate ones with the sin/cos transformation in order to capture the temporal proximity. Furthermore, each of these features were nromalized in the range [0, 1].  

### 5.4. Miscellaneous  
From the correlation matrix (Figure 21), we notice that only temperature and apparent temperature have high correlation coefficient. In order to avoid redundant features in our datasets, we discard apparent temperature.  

The train/val/test split was 90%, 5%, 5%. In order to avoid look-ahead bias, the partitions are consecutive.  

Even though we tried our best to discard the features and periods with a lot of missing values, we still have missingness in our data. In order to cope with this problem, several imputation techniques were utilized.  The process of evaluating the imputers was as following:
1. We fit each imputer on the train set.
2. We generate probability mass function for each feature to have a missing value, based on the data of the train set
3. We discard row of the validation set with missing values
4. We iterate the set generated in step 3 and purposefully discard some of the values, with probabilities defined in step 2 (the discarding is not performed in place - we create a copy of the dataset and make the discards there). In this moment, we have two matrices: one with missing values (step 4) and another with the true values (step 3)
5. Using the fitted imputers (step 1) we fill in the missing values from the matrix in step 4.
6. We calculate the distance between the imputed matrix (step 5) and the matrix with true values (step 3) using Frobenius norm.   

The results are summarized in Table 1. Note: it was impossible to fit certain imputers due to RAM shortage. 

The missing values for each dataset were filled with the accordingly best performing imputer. Before filling in the values, for each of the pollutant we added an additional feature indicating the whether the value was imputed or not.  

Finally, we are ready to construct the datatsets in a format that is appropriate for the seq2seq models. Each dataset is further split into:
* Encoder input data - a tensor of shape (num_samples, T<sub>x</sub>, num_encoder_features), where T<sub>x</sub> is the input sequence length. Air pollution, weather and datetime features are passed as input for the encoder. These features correspond to the historic timesteps.
* Decoder input data - a tensor of shape (num_samples, T<sub>y</sub>, num_decoder_features), where T<sub>y</sub> is the output sequence length. Date time features, PM<sub>10</sub> predictions in the previous step, and the context vector in case of an attentive architecture are passed as input for the decoder. These features correspond to the future timesteps. Note: we can generate the datetime features for the future timesteps beforehand since they are deterministic, hence why we can use them as input for the decoder without suffering a data leak. 
* Decoder target data - a tensor of shape (num_samples, T<sub>y</sub>, 1) since we are predicting one PM<sub>10</sub> only. Note: we discard all samples which have at least one imputed PM<sub>10</sub> value in the target vector. 

## 6. Modeling
As we mentioned before, we are going to compare the performances of two seq2seq architectures: standard vs attentive. At this point it is fair to note that the input sequences are 24 hours long and we are trying to forecast the next 12 hours of PM<sub>10</sub> pollution (output sequence length). We use the following notation for model description:
* x<sup>t</sup> - vector consisting of air pollution, weather and datetime features at time t. This is an input for the encoder LSTM cells.
* z<sup>t</sup> - vector consisting of datetime features only at time t. This vector concatenated with the PM<sub>10</sub> predictions in the previous step, y<sup>t-1</sup>, and the calculated context vector c<sup>t</sup> (in case of an attentive architecture) is used as an input for the decoder LSTM cells. 
* y<sup>t</sup> - output vector at time t.

Both models (Fig 22 and 23) are implemented in Tensorflow 2.1.0 through the Keras API. We used MSE as the loss function with the Adam optimizer. The models were trained on several GPUs offered by Google Colab. In order to save time training unsuccessful models, we used the Early Stopping criteria with the patience parameter varying between 10 and 15.   

In order to find the best hyperparameters for both architectures, a random search was performed. The hyperparameter ranges were based on [Klaus et al. (2015). LSTM: A search space odyssey. IEEE transactions on neural networks and learning systems. 28. 10.1109/TNNLS.2016.2582924] and empirical research. Moreover, the ranges were constantly updated based on the top 10 performing models for each architecture in order to narrow down the search space. 

## 7. Results  
The best models from the two architectures were evaluated on the test sets using the following metrics: RMSE and R<sup>2</sup>. RMSE is a metric that indicates how much the predictions differ from the true values. On the other hand R<sup>2</sup> indicates what proportion of the variance  of the dependent variable is predicted by the model. A model that predicts the mean only (without taking into account the inputs) will have R<sup>2</sup>=0, while a model that perfectly predicts the target values will have R<sup>2</sup>=1. When using non-linear models, it is also possible that R<sup>2</sup><0. In this case, predicting the mean will yield better results.  

From the results in Table 4, we can notice that the attentive architecture consistently outperforms the standard one. However, when comparing the architectures, we should note that the choice of metric is very important. For illustration, upon observing the results for the test set of Miladinovci, we notice that the improvement in the RMSE score goes from 40.641 to 32.906 (nearly 8 points). This difference may not seem very significant, but if we notice the improvement in the R<sup>2</sup> score (from 0.239 to 0.501), we see that there is an improvement of nearly 30% of the explained variance of the variable we are predicting, which is siginificant. This is true for all other datasets, except for Lisice.  

First of all, the reason why we have such a low RMSE score for the test set of Lisice is because it largely contains the summer period (from 19.06.2018 till 03.10.2018, Table 2), period when the air pollution is lowest (Figure 12). Furthermore, we notice that the dummy model that predicts the mean only is almost as good as the trained models - again, this is as a result of the smaller variance in the pollution during the summer period (Figure 24), hence why the mean is a good estimator.  

Naturally, we notice that predicting the mean for the other test sets (which largely contain the winter period) give significantly worse results (ex. Karpos, RMSE=50.214 for the dummy model vs RMSE=34.660 for the attentive model) since the variance is larger (Karpos, Figure 24).

## 8. Future work
Lastly, we propose several ideas for future work which could potentially improve the model performances:
1. Predicting all of the available pollutants for each dataset. By propagating the predictions for the other datasets as an input for the decoder, the model could perform better since the air pollution features are highly correlated between themselves.
2. Appending weather forecast as an input in the decoder. Due to the relationship between the air pollution and weather, the model performance could be boosted if it has an estimate of the weather condition for the future timesteps. 
3. Utilizing GRU instead of LSTM cells. These cells have significantly simpler architecture which could lead to models that generalize better. 

We have started working on implementeding proposals 1 and 2 in an automated manner. The files are located in the 'automation' directory in this repo. 

## Conclusion
The World Health Organization indicates the air pollution as a hazardous byproduct of human activity, causing serious health problems, accounting for over 4.2 million premature deaths worldwide [World Health Organization. (‎2018)‎. Air pollution and child health: prescribing clean air: summary. World Health Organization. https://apps.who.int/iris/handle/10665/275545. License: CC BY-NC-SA 3.0 IGO]. Moreover, in a recent study researchers from Harvard showed that an increanse of only 1 μg/m<sup>3</sup> of PM<sub>2.5</sub> is associated with an 8% increase in the COVID-19 death rate (95% confidence interval) [Exposure to air pollution and COVID-19 mortality in the United States. Xiao Wu, Rachel C. Nethery, Benjamin M. Sabath, Danielle Braun, Francesca Dominici. medRxiv 2020.04.05.20054502]. Having this in mind, we turn to forecasting the air pollution level with the goal of contributing to the research and projects developing tools that people use in order to limit their outdoor exposure to these harmful particles. We compared the performances of two seq2seq architectures and demonstrated the superiority of the attentive models on datasets stemming from 5 air pollution sensors in Skopje, North Macedonia. However, a true change will not occur when someone makes the most correct forecasting model, but when the institutions in power, in collaboration with the citizens, will take the appropriate measures to reduce the pollution.














