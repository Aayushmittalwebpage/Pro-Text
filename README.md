# Pro-Text
Sequence tagging for textualisation process.

There are total 4 model used.

Model Description:-

MODEL 1 (No. of pauses / sentence)

Data - 1)  First I preprocessed the data into sentences and no. of times there are pauses in these sentences. eg. 

                 
      f out out sortie § 3
                   
      Dans l' Arc tiquel ' aperçois réalise 5


2)

a)Created Tabulardataset set using torchtext ({'text': ['f', 'out', 'out', 'sortie', '§'], 'label': '3'}) 

c) Then Builded vocabulary for the text and convert them into integer sequences

Builded vocab of embeddings using glove.6B.100d embeddings on the train data.

3) Created batches for training the model of size 64.

4) 

a)Created model using an embedding layer, Bi-lstm layer and a linear layer.

Model summary 
Regressor(
  (embedding): Embedding(5573, 100)
  (lstm): LSTM(100, 32, num_layers=2, batch_first=True, dropout=0.2, bidirectional=True)
  (fc): Linear(in_features=64, out_features=1, bias=True))


b) Used Adam optimizer with a learning rate of 0.001 and calculated the MSE loss for the train and valid data.

5) Train on 50 epocs and calculated R-squared value on test data and got 0.42.








MODEL 2 ( Input: word, Output: ‘P’ / ’_’ )

Preprocessed data into [(['out'], ['P']), (['sortie'], ['_'])...] 
Created a dictionary of words to index. Replace target ‘P’ with 1 and ‘_’ with 0.
Model - Embedding layer with output of 64 dimension, lstm layer and a linear layer.

LSTMTagger
(
  (word_embeddings): Embedding(6030, 64)
  (lstm): LSTM(64, 64)
 	 (hidden2tag): Linear(in_features=64, out_features=2, bias=True)
)

	
Used SGD optimizer with 0.001 learning rate and calculated negative log likelihood loss and train on 80% data.

Ran 5 epochs but the loss is not decreasing at all with increase in epocs. Most of the predictions are ‘_’. Got accuray of around 84%, but I don't think the model is learning anything as there are very few ‘P’ in the predictions and mostly are ‘_’.


MODEL 3 (Input: word in a sentence, Output: Corresponding  ‘P’/’_’ tag) (Bi-LSTM with CRF) (No-pretrained embedding)

Preprocessed data sample - [('f', '_'), ('out', 'P'), ('out','P'), ('sortie', 'P'), (' §', '_')], ..

Preprocess further by converting each text word to a corresponding integer ID. To feed the data into the model, convert each sentence into the same length by setting a threshold value of 75 words and adding smaller sentences with a word ‘PAD’ and trimming the longer sentences.
	
Introduce a 3rd target category called ‘pad’ which corresponds to the padded word in the sentences. So target : {0: 'PAD', 1: '_', 2: 'P'}

Model 

Layer (type)                 Output Shape              Param #   
=================================================================
input_1 (InputLayer)         (None, 75)                0         
_________________________________________________________________
embedding_1 (Embedding)      (None, 75, 300)           1429800   
_________________________________________________________________
bidirectional_1 (Bidirection (None, 75, 100)           140400    
_________________________________________________________________
time_distributed_1 (TimeDist (None, 75, 50)            5050      
_________________________________________________________________
crf_1 (CRF)                  (None, 75, 3)             168         
Embedding dimension - 300, batch size - 32 optimizer - rmsprop. Trained 90% data with 5 epochs. (val_loss was decreasing until 5 epochs).

Evaluation 

             precision    recall  f1-score   support

           P       0.55      0.26      0.36       668
         PAD       1.00      1.00      1.00      9079
           _       0.86      0.95      0.90      3078



MODEL 4 (Input: word in a sentence, Output: Corresponding  ‘P’/’_’ tag) (Bi-LSTM with CRF) (Glove embedding)

Everything is similar to Model 3. The only change is; instead of using random initialisation of weights in model I have used pretrained glove embeddings of 300 dimension.

Result

             precision    recall  f1-score   support

           P       0.49      0.22      0.30       674
         PAD       1.00      1.00      1.00      8755
           _       0.86      0.95      0.90      3396

