# LSTM Music Generation ♫♪♪
Marlon Grandy, Justin Zheng, and Oliver Wang
## 1. Introduction
As a group of avid music listeners and some musicians, we thought it would be interesting to utilize what we learned in the past three months to see if could generate some sort of music using a database. Our project goal was to train a LSTM (Long Short-Term Memory) artificial neural network that could generate classical music. The initial hypothesis was less fruitful than the outcome. We believed that the model would only be capable of creating "background" style music, or elevator music, that resembled classical styles. However, as you will see later in the blog, our expectations were superseded.

**Why LSTM?**

To understand LSTM's, we should first understand recurrent neural networks (RNN). Much like human thinking, RNNs utilize previously encountered data for the current task instead of just forgetting about them like traditional neural networks. In essence, recurrent neural networks are loops which allow information to persist. As a result, RNNs have been successfully used in problems such as speech recognition, language modeling, translation, and image captioning. For more information regarding RNNs feel free to consult [Andrej Karpathy's blog post](http://karpathy.github.io/2015/05/21/rnn-effectiveness/).

Long Short-Term Memory neural networks don't have certain problems that traditional recurrent neural networks have, specifically the long term dependency problem. LSTMs were designed to avoid this problem and "remembering information for long periods of time is practically their default behavior." (Colah's Blog). Without getting too in depth, the main idea of LSTMs is the cell state which runs along the entire network and allows for easy transport of data to flow unchanged. You can read more about the intricacies at [Colah's Blog](https://colah.github.io/posts/2015-08-Understanding-LSTMs/). This attribute of LSTMs was particularly attractive and important to solving our problem. Because music is sequential and the current note may depend on previous notes far in the past (say 8 or 16 bars), LSTM networks excel in these exact conditions. The neural network must capture time dependent relationships between notes and chords.

**Project Overview**

<img width="768" alt="Screenshot 2022-12-09 at 11 21 59 AM" src="https://user-images.githubusercontent.com/109930628/206681505-8c6d60b6-a5a6-4cdd-9425-58e047fee4ea.png">
We trained three models from the dataset: notes, offsets, and durations, that were later combined to form our new Midi file. To process the data, we utilized [Music21](https://web.mit.edu/music21/doc/about/what.html), a simple but powerful toolkit for computer-aided musicology, developed by MIT, Harvard, Smith, and Mount Holyoke Colleges.

## 2. Dataset
We chose a [kaggle dataset](https://www.kaggle.com/datasets/soumikrakshit/classical-music-midi) of classical music midi data as the data for training the music generation mode. Tha data was initially scraped from [this](http://www.piano-midi.de) site containing piano midi files. The dataset includes 290 songs, 19 composers, and 550,000 notes in total. When training our model, we used subset of 45 songs, 5 composers, and 75,000 notes.
## 3. Preprocessing
Preprocessing was arguably the most important step for us. We wrote a function that used Music21's "converter.parse" method to read in each file. Every note and its metadata, was then separated and added to notes, offsets, and durations. Given that we were parsing piano music, we also had chord recognition functionality that treated notes slightly differently to accomodate. Offsets were then converted to categorical in the sense that the note's offset was relative and dependent on the previous note's offset. For example, a note with an absolute offset of 2 and a previous offset of 1 would be converted to 1 (2-1 = 1) as it is 1 offset away from the previous note. The data was later one hot encoded to determine what note it was. For training, our data was read at a segment length of 64 which we decided on after experimenting with many different values. In essence, we are reading 64 notes at a time and then shifting the window forward by one note each iteration. Lastly, we made an output vector with the targeted values.

## 4. Model Architectures
Throughout the progression of the project many different model architectures were tested. Initially, a single model was trained on just the note data. This model architecture resulted in a training accuracy of 43% and a validation accuracy of 37%. As the project progressed, and the corresponding note durations and offsets were added as data, the model architectures also progressed.

<img width="632" alt="Screen Shot 2022-12-08 at 6 46 51 PM" src="https://user-images.githubusercontent.com/106160715/206667768-f41ff532-9443-46a8-b79d-efd87b177fae.png">

The second model architecture that was tested was built to predict on just the note and offset data. The offset data is initially a regression problem and the notes a classification problem. In order to avoid training two models, a network architecture inspired by Jason Brownlee’s post on “Neural Network Models for Combined Classification and Regression” was constructed. The architecture contained two output layers: one for a target regression array and one for a target classification array. In input to this network was a one hot encoded vector with the note offset value tacked onto the bottom. The hope was that, given enough data, the network would pick up on the offset values for the regression problem and the one hot encoded values for the classification problem. While the outputs for this model were not saved due to the model doing exactly the opposite of what was needed to happen, in short the accuracies for the classification problem were very poor. Furthermore, the regression outputs were not in increasing sequential order, so notes did not follow the right order when read back into MIDI format.

<img width="522" alt="Screen Shot 2022-12-08 at 6 59 51 PM" src="https://user-images.githubusercontent.com/106160715/206667896-b7576cbc-f554-40a5-888c-43419c8f9d23.png">

Inspired by the previous architecture, a new model was created. We still wanted to try and predict the notes and note offsets within a single model. However, this time we attempted to frame the problem in terms of multilabel classification. If we squeeze the one hot encoded note arrays together with the one hot encoded offset arrays (as discussed in the preprocessing section) into a single array, maybe we could simultaneously classify the categorical notes and offsets and present the project as a multilabel classification problem. However, similar to the previous model the accuracies for both offsets and notes were very low. Furthermore, offsets were sometimes classified as chords/notes and chords/notes were sometimes classified as offsets. Overall, not very effective for generating incredible classical music.

<img width="522" alt="Screen Shot 2022-12-08 at 6 59 51 PM" src="https://user-images.githubusercontent.com/106160715/206668314-815d0c5f-1eb2-4d69-9f54-06407a58c698.png">

At this point, even with no prior success with offsets and notes, we wanted to add in note durations to the picture (always dream big). With a history of failure we knew we had to do something different… something bold. The single model for all metadata has shared weights for all the metadata, potentially uncovering relationships of what makes a note go well with some offset and some duration. However, because we are sequentially modeling the data, we could theoretically split up the data into three different models and still uncover what notes go well with what offsets and durations. Resultantly, we ended up training three models for the notes, offsets, and durations. After many attempts of fine tuning, we were able to predict notes, durations, and offsets using a single model architecture.

<img width="628" alt="Screen Shot 2022-12-08 at 7 40 26 PM" src="https://user-images.githubusercontent.com/106160715/206668068-44ec1e2a-d54a-4261-bdda-936638dc1aed.png">

The default model.compile parameters throughout the model architecture experimentation process are shown below: 

<img width="349" alt="Screen Shot 2022-12-08 at 7 46 16 PM" src="https://user-images.githubusercontent.com/106160715/206668155-a4f622fc-e4d1-4191-b636-479bd1a79f7a.png">

## 5. Model Performance
The training and validation accuracies for notes, durrations, and offsets are shown below:

<img width="1000" alt="Screen Shot 2022-12-09 at 10 22 09 AM" src="https://user-images.githubusercontent.com/106160715/206668624-1d57b604-0ce3-4336-b9fa-ce83ac66bf0b.png">

The note model had the lowest validation accuracy of 63%. The offset model had a validation accuracy of 87%. Finally, the duration model had a validation accuracy of 88%. The number of classes being predicted for the notes (277) was far greater compared to the offsets (28) and durations (36). Resultantly, the smaller notes accuracy was logical. All of the validation losses and accuracies were less than the training losses and accuracies. After experimenting with different model architectures to deal with the faster training progressions, we found that a dropout layer with a dropout rate of 0.25 was the most effective solution. 

## 6. Complications
Over the course of the project we faced a few issues that we needed to work through:
### Excessive note repetitions
Over previous iterations of our model, the generated notes entered into cycles where the same few notes were generated over and over without any interesting variation. We were able to resolve this by adding a diversity function to sample the note probability array instead of taking the largest probability value. The diversity function allowed the model to exit from the repetitive note cycles. The diversity function is shown below:
<img width="608" alt="Screen Shot 2022-12-09 at 10 48 01 AM" src="https://user-images.githubusercontent.com/106160715/206673970-d7714d78-7df4-458d-ba36-42dfff36153f.png">

### Generating off of a bad seed
The seeds used to generate music were 64-note segments picked at random from our dataset. At times, we chose seeds that sounded, to say the least, very bad. These seeds could have been from the start or end of a song, or simply a particularly boring segment. However, there were also times when the seed sounded good but the generated music sounded bad. This was likely due to the data the model was trained on and an inability for the model to generalize to the seed it was given.

### Lack of RAM available in Google Colab
Unfortunately, we were limited by the technology we had available to us. This led to training the model on a subset of the full dataset. For deep learning models, more data will likely lead to better performance. In order to use as much data as possible we implemented garbage collectors whenever a variable became obsolete. There were other ways that we could have optimized the code to use less RAM, and it would be interesting to see how much better the model could be given more data. 

## 7. Music Samples
Found below are samples of music that we were able to generate with our network. The first audio file is the seed that we fed the network, and the following one is the resulting output.

https://user-images.githubusercontent.com/82842273/206411747-683e7858-cc00-46b6-be91-ed5250be1665.mp4


https://user-images.githubusercontent.com/82842273/206411764-4d547ddf-cfd8-4b3f-902e-9ea94ede26a8.mp4

## 8. Closing Thoughts

