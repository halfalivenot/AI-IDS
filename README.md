# AI-IDS

## Stage 1: Data collection and dataset creation  
•	Set up a Windows Server virtual machine on GCP  
•	Pipelined logs into BigQuery studio  
•	Collected events over a 3-day period  
•	Organized the raw Window logs into sessions  
•	Merged with another dataset to balance out the malicious and benign traffic  
•	Final dataset contains 10,645 records and six columns:  

| Columns | Purpose |
|----------|----------|
| session_id | Unique identifier for each session |
| login_attempts | Number of login attempts |
| session_duration | Duration of the session |
| ip_reputation_score	| Reputation score associated with the source IP | 
| failed_logins |	Number of failed login attempts |
| attack_detected	| Target classification (0 or 1) |

We collected Windows events from a honeypot environment and transformed them into a structured session-based dataset. We selected features that could help identify suspicious activity, such as login attempts, session duration, failed logins, and IP reputation. Each session was also assigned a binary classification indicating if an attack was detected.

## Stage 2: Feature selection and preprocessing
How we prepared the dataset for the machine learning algorithms.  
Separating the dataset into two parts:  
•	X: The input features used to make predications  
•	y: The target variable the model is trying to predict  
The input features were:  
•	Login attempts  
•	Session durations  
•	IP reputation score  
•	Failed logins  
The target variable was attack_detected. We dropped the session_id because it was an identified, rather than a meaningful predictive feature.   
We standardize the data using StandardScaler from Scikit-learn. Standardization adjusts numerical features to a comparable scale, generally giving them a mean of zero and standard deviation of one. This is useful for neural networks because features with significant different numerical ranges can affect the learning process. However, we only fitted the scaler using training data only, then applied the same transformation to validation and testing data. This helps prevent data leakage.   

## Stage 3: Splitting the dataset
We divided the 10,645 records into three groups.   
•	Training: Used to teach the model patterns in the dataset  
•	Validation: Used to monitor model performance during training  
•	Testing: Used to evaluate the model on data it did not train on.   
Using stratify when splitting the data helps to preserve the proportion of attack and non-attack records across the different subsets.   

## Stage 4: AI-IDS model
### Model A: Baseline Artificial Neural Network (ANN)  
The primary deep-learning model developed using TensorFlow and Keras.   
•	Four input features: login attempts, session duration, ip reputation, and failed logins  
•	Hidden layer 1: 64 neurons  
•	Hidden layer 2: 36 neurons  
•	Hidden layer 3: 16 neurons  
•	Output layer: 1 neuron   
The hidden layers are responsible for learning patterns and relationships between the input features. Using ReLU in the hidden layers, it introduces non-linearity, allowing the model to learn more complex relationships. We used Sigmoid activation in the output layer because the task is binary classification.   
The Adam optimizer adjusts the mode’s weights during training to minimize its predication error. Binary cross-entropy measure the differences between the model’s predicted probabilities and the actual binary labels. An epoch represents one complete pass through the training dataset.   

### Model B: Random Forest Classifier   
We’ve also developed a Random Forest Classifier using Sckit-learn. Unlike an ANN, Random Forest uses an ensemble of decision trees.   
The general process is:  
1.	Multiple decision trees are trained using the training data.  
2.	Each tree learns different decision patterns  
3.	Each tree makes its own prediction  
4.	The predications are combined through voting to produce the final classification  
We configured the initial Random Forest model with:  
•	100 decision trees (n_estimators=100)  
•	Random state of 42 for reproducibility  
•	Balanced class weighting to account for differences in class distributions   
Including a Random Forest is meant to compare our neural network. If the additional complexity will provide an advantage.   

### Model C: Experimental ANN
We also experimented with a second neural network to see how different configurations affected performance.  
| Parameter | Baseline ANN | Experimental ANN |
|-----------|------------|------------------|
| Hidden layers	| 3 |	2 |
| Neurons	| 64, 36, 16 |	32, 16 |
| Optimizer	| Adam | SGD |
| Epochs	| 10	| 5 |
| Output | Sigmoid | Sigmoid |
The experimental version is smaller and uses SGD instead of Adam. It is meant to allow us to investigate whether changing the network architecture and optimization method would affect its learning performance.   

### Stage 5: Evaluation
Model accuracy comparison  
•	RF: 87%   
•	Baseline ANN: 86.20 %  
•	Experimental ANN: 66.01%   
RF performed slightly better than the baseline ANN in overall accuracy. However, accuracy isn’t the only metric we’re looking at.   
Precision, recall, and F1-score  
Precision:  When the IDS identifies something as an attack, how often is it actually an attack?   
Recall: Out of all the actual attacks, how many did the IDS successfully identify?   
F1-score: Combines precision and recall into one measurement using their harmonic mean   

| Metric | Baseline ANN | Random Forest |
|-------|------------------|----------|

The RF achieved better recall and F1-score than the baseline ANN, while the ANN achieve higher precision. The best model will depend on what an IDS should prioritize. For example, low precision could mean legitimate activity is being falsely identified as malicious while low recall means genuine attacks are being missed.   

Confusion matrices
| Outcome | Meaning |
|----------|-------|
