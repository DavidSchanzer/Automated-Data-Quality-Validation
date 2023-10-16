# Automated-Data-Quality-Validation
A Python-based data quality classifier of newly-arriving tabular data files into "trusted" and "suspect" categories using Novelty Detection machine learning.

Based on the work of Redyuk et al. (2021) and the GitHub repo at https://github.com/sergred/automating-data-quality-validation-data, this repo allows for the prototyping of this approach to automated data quality validation and the extension of this research by:
1.	testing its applicability to a real-life dataset;
2.	optimising its performance through algorithmic tuning and testing alternative algorithms;
3.	providing guidance on why rejected files were considered suspect.

This approach is applicable to any scenario in which externally supplied tabular data files are regularly provided to an organisation, and the organisation needs to ensure that the data quality of newly arriving files has not degraded compared with previously received files.

The novelty of the solution proposed by Redyuk et al. (2021) was that:
1.	It requires no data domain expertise because past data from each external data feed is used to train a novelty-detection machine learning model that learns what “normal” data looks like.
2.	It learns and incorporates the acceptable and expected changing data patterns, such as Year Of Birth increasing over time, by re-fitting the machine learning model with each file that is considered acceptable.
3.	It allows for alerts to Data Quality staff when the data quality in a new file differs significantly from its learned definition of “normal” data. The Data Quality staff can then inspect the rejected file and determine whether the file was validly or invalidly rejected.

The solution has been coded in Python, and uses the PyOD (Python Outlier Detection) module in order to test the Novelty Detection algorithms. All data used for training and testing is considered to be of acceptable quality (i.e. “clean” data), given that it has previously been received and ingested into the organisation’s data warehouse.

However, since Novelty Detection in machine learning is a One-Class Classification problem because only “clean” data is available, there is therefore a need for a means to evaluate the model’s ability to correctly distinguish “clean” files from “dirty” files (ie. those that contain data quality errors). This is achieved by generating, for each “clean” file, an equivalent “dirty” file by injecting synthetic errors into between 1% and 80% of the records in the file, and using one of the following six types of synthetic error:
1.	Explicit missing values (empty cells)
2.	Implicit missing values (cells that are logically empty but contain 99999 for numeric fields or NONE for textual fields)
3.	Numeric anomalies (unexpectedly high or low numeric values)
4.	Swapped numeric fields (swapping values between two numeric attributes)
5.	Swapped textual fields (swapping values between two textual attributes)
6.	Typographic errors (randomly replacing a fraction of letters in textual attributes with adjacent letters on a QWERTY keyboard, simulating “butterfinger” typing)

The available “clean” and “dirty” files are then split into a Training set and Testing set, and the model is trained on a set of descriptive statistics calculated on each of the “clean” Training set of files.

Once the Training phase has been completed, the algorithm iterates through the Test files, randomly choosing either a “clean” or a “dirty” file each time, and calculating the same set of descriptive statistics. The chosen Test file is then classified by the model as either an “inlier” (i.e. its descriptive statistics are sufficiently similar to the data in the Training data) or an “outlier” (i.e. its descriptive statistics are sufficiently different to the data in the Training data).

The results of this classification can then be recorded as one of these four values:
1.	A True Negative (TN), being a clean file that’s been classified as clean.
2.	A True Positive (TP), being is a dirty file that’s been classified as dirty.
3.	A False Negative (FN), being a dirty file that’s been classified as clean.
4.	A False Positive (FP), being a clean file that’s been classified as dirty.

If a file receives a TN or FN result, the file is used to re-fit the model so that the data in this file is included in the model, which therefore modifies its definition of “normal” data to include the latest data.

If a file receives a TP or FP result, the equivalent “clean” file is used to re-fit the model so that the data in the clean version of this file is included in the model, which therefore modifies its definition of “normal” data to include the latest “clean” data.

_Redyuk, S., Kaoudi, Z., Markl, V., & Schelter, S. (2021, March 23-26). Automating data quality validation for dynamic data ingestion [Paper presentation]. 24th International Conference on Extending Database Technology, Nicosia, Cyprus.
_
