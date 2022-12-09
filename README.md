# LSTM Music Generation ♫♪♪
Marlon Grandy, Justin Zheng, and Oliver Wang
## 1. Introduction
## 2. Dataset
We chose a [kaggle dataset](https://www.kaggle.com/datasets/soumikrakshit/classical-music-midi) of classical music midi data as the data for training the music generation mode. Tha data was initially scraped from [this](http://www.piano-midi.de) site containing piano midi files. The dataset includes 290 songs, 19 composers, and 550,000 notes in total. When training our model, we used subset of 45 songs, 5 composers, and 75,000 notes.
## 3. Preprocessing
## 4. Model Architectures
Throughout the progression of the project many different model architectures were tested. Initially, a single model was trained on just the note data. This model architecture resulted in a training accuracy of 43% and a validation accuracy of 37%. As the project progressed, and the corresponding note durations and offsets were added as data, the model architectures also progressed.
[inset image]
The second model architecture that was tested was built to predict on just the note and offset data. The offset data is initially a regression problem and the notes a classification problem. In order to avoid training two models, a network architecture inspired by Jason Brownlee’s post on “Neural Network Models for Combined Classification and Regression” was constructed. The architecture contained two output layers: one for a target regression array and one for a target classification array. In input to this network was a one hot encoded vector with the note offset value tacked onto the bottom. The hope was that, given enough data, the network would pick up on the offset values for the regression problem and the one hot encoded values for the classification problem. While the outputs for this model were not saved due to the model doing exactly the opposite of what was needed to happen, in short the accuracies for the classification problem were very poor. Furthermore, the regression outputs were not in increasing sequential order, so notes did not follow the right order when read back into MIDI format.
[inset image]
Inspired by the previous architecture, a new model was created. We still wanted to try and predict the notes and note offsets within a single model. However, this time we attempted to frame the problem in terms of multilabel classification. If we squeeze the one hot encoded note arrays together with the one hot encoded offset arrays (as discussed in the preprocessing section) into a single array, maybe we could simultaneously classify the categorical notes and offsets and present the project as a multilabel classification problem. However, similar to the previous model the accuracies for both offsets and notes were very low. Furthermore, offsets were sometimes classified as chords/notes and chords/notes were sometimes classified as offsets. Overall, not very effective for generating incredible classical music. 
[inset image]
At this point, even with no prior success with offsets and notes, we wanted to add in note durations to the picture (always dream big). With a history of failure we knew we had to do something different… something bold. The single model for all metadata has shared weights for all the metadata, potentially uncovering relationships of what makes a note go well with some offset and some duration. However, because we are sequentially modeling the data, we could theoretically split up the data into three different models and still uncover what notes go well with what offsets and durations. Resultantly, we ended up training three models for the notes, offsets, and durations. After many attempts of fine tuning, we were able to predict notes, durations, and offsets using a single model architecture.
[inset image]
The default model.compile parameters throughout the model architecture experimentation process are shown below: 
[inset image]

## 5. Model Performance
The training and validation accuracies for notes, durrations, and offsets are shown below:
[inset image]
The note model had the lowest validation accuracy of 63%. The offset model had a validation accuracy of 87%. Finally, the duration model had a validation accuracy of 88%. The number of classes being predicted for the notes (277) was far greater compared to the offsets (28) and durations (36). Resultantly, the smaller notes accuracy was logical. 
## 6. Complications
Over the course of the project we faced a few issues that we needed to work through:
### Excessive note repetitions
Our network used to get into cycles where the same few notes were played over and over without any interesting variation. We were able to resolve this by adding diversity functions to our sampling which allowed for more interesting notes to be included.
### Generating off of a bad seed
Generally, our seeds are 64-note segments picked randomly from our dataset. At times, we could get a poor seed by picking the start or end of a song, or a particularly boring segment. When this occurs, our generated audio is impacted heavily.
### Lack of RAM available in Google Colab
Unfortunately, we were limited by the technology we had available to us and were not able to train using the entire dataset. We then had to resort to using only a subset of the artists, which was still successful but not ideal.
## 7. Music Samples
Found below are samples of music that we were able to generate with our network. The first audio file is the seed that we fed the network, and the following one is the resulting output.

https://user-images.githubusercontent.com/82842273/206411747-683e7858-cc00-46b6-be91-ed5250be1665.mp4


https://user-images.githubusercontent.com/82842273/206411764-4d547ddf-cfd8-4b3f-902e-9ea94ede26a8.mp4

## 8. Closing Thoughts

