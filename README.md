# **Doccurate: A Curation-Based Approach for Clinical Text Visualization**
```
                            Sai Vamsi Ogety
                             MS in CE(CS)
                                CIDSE
                       Arizona State University
```
## **INTRODUCTION**

Electronic Medical records uses text as their preferred mode of documentation since it preserves the context while explicitly communicating the sequences of medical events. It even gives the flexibility to support doctors documentation needs. This power of text falls once the scale of text goes from few sentences to hundreds of documents which makes it difficult in getting the overview of a patient. Physicians have very little time to grasp the overview of the patient and this may lead to patient safety issues. To overcome this issue, a notable research has been done to obtain the overview of patient medical records using data visualization and text summarization. Fully automated text processing is a powerful tool in text visualization but it has few challenges. First, the automatic processing errors need to be identified and acknowledged. Second, Doctors have unique needs depending on various factors like their speciality, patient’s case and their own way of mapping things from the documents. Medical records can be organized in different ways based on time, source and problem-oriented all these types serve different clinical needs. Since there are different ways to organize the data working with predefined structure is not a solution to our problems we discussed before. Even when we accommodate the flexibility of control to users for automated tasks doctors do not have the expertise or time to fine tune the model according to their needs. These issues should be considered as part of integral part of user experience and should be incorporated into the design. The motivation for the approach in this paper is taken from visual curation where visualization aids the user-in-the-loop iterative refinement of automated process. It also provides reusable and customized doctor defined filters defined by medical taxonomies which helps in aggregating information from pre-processed text. The key details of the automated output are verified and communicated transparently. We present *Doccurate,* a clinical text visualization prototype which helps in visualizing a large number of patient records. Results cover case studies on two patients dataset and how it explains the summary of their medical history in a short period of time and the also the extension proposed to the original system.

### **KEYWORDS**

Text processing, Text visualization, Filtered Collection, Medical taxonomy.

## 1. **Visualization design**

The main components of the system include the Control Panel, Timeline and the Text Panel (Figure 2).

The Control Panel contains the patient related information name, age and gender along with document filters for binning time interval and filter collections.

![](/images/image2.png)![](/images/image1.png)![](/images/image3.png)

> Fig 1. Timeline view with three levels of detail - Category, Diseases and Symptoms

Filter collections are a set of related concepts that are applicable to a shared meaning that come under a broader umbrella. \[1\] These are basically physician defined filters to get a clear view of the clinical content. These filters help populate the timeline. The timeline provides an overview in a time-oriented manner of the selected filter collections.

![](/images/image6.png)

> Fig 2. Original system’s interface

In addition to this the Timeline contains a sorted view of all the items that are present in the current level. There are three levels that are displayed in the timeline section. In the first level all the filter collections are displayed along with the frequencies of the sub parts of these filter collections. In the second level all the tags that are related to the filter collection are displayed by actually selecting the filter collection. The third level is displayed by clicking on the tag, this gives a more detailed view of the content.

This also populates the Text Panel which contains the actual snippet of the passage present in the patient’s record. At all levels the frequency stream is displayed along with the encompassed text. This Text Panel has all the documents as mentioned, along with this the terms that are related to the selected filter collections are highlighted to get a better visual representation of the medical records. Once a medical record is displayed a horizontal line in the timeline is shown, this represents the actual time when the document was recorded.

### 1.1 **Implementation**

Our interface contains the **Control Panel** which contains the patient demographics along the filter collections. These filter collections are the broader umbrella of diseases. The **Timeline Panel** initially displays the frequency of diseases occurred in each of these filter collections. This generates a violin timeline visualization. Moreover, the interface contains a sorted view of all the filter collections. This can be effectively used by doctors(users of this interface over here) to get a high level view of all the diseases that the patient encountered with the specific timeframe. This just gives an overview of all the diseases. So, to get an in depth view for further analysis between the disease and the symptoms the user can select one of these diseases which in turn generates another violin timeline visualization of the frequencies of symptoms. Along with the sorted view of the diseases is displayed. Here the user can see the actual symptoms that caused the disease in a specific timeframe. This visualization will be helpful to doctors as they can relate the symptoms observed by the patients to the symptoms seen in the visualization and detect any potential disease. This will guide the doctors towards prescribing medicines and even starting a new treatment.

![](/images/image5.png)

> Figure 3. Implemented system interface

In addition to this, if the user selects a given filtered collection from the violin chart it opens up new chart with corresponding diseases related to it this will aid doctor to quickly check what are the diseases that are most frequent in the patient medical history and also gives the information about any medical condition that the patient had in his/her history. We can gain even deeper understanding about any of the diseases at the current level by selecting one of them. This selection will take to the final level where it gives information about the related symptoms and their frequency, this level aids in understanding what are the common symptoms that a patient had and also helps in deciding to whether to continue with the previous treatment if the observed symptoms are same as previous or create a new treatment plan if any new symptoms were observed.

The **Text Panel** displays the patient's medical records with the symptoms and keywords highlighted and the documents displayed corresponds to the filtered collections, diseases and symptoms that are observed on the violin chart.

## 2. **Extension**

The implementation system takes into account the filter collections as a broader umbrella of diseases.

![](/images/image4.png)

> Fig 4. Tree visualization

Along with this a level wise deeper analysis of diseases and symptoms can be done through the violin frequency visualization. Although the visualizations that are implemented give a good in-depth perspective of the diseases and symptoms related to the specific patient there is no visualization of the actual taxonomy itself.

We have implemented an extension by creating a tree visualization of the filter collections, diseases and symptoms. The filter collections are considered as root of the broader category. The diseases that fall in this category are it’s children. In addition to this, these diseases have symptoms as it’s children. So, a proper hierarchy of the diseases is visualized in the tree which provides a holistic picture of not only the patient's health problems but also will be useful for doctors to analyze the diseases for patients and their relation with many other symptoms and potentially other diseases as well.

## 3. **Case Studies**

We have generated two patient datasets that contain the patient encounter data which contains the encounter ID, the start and end date of this encounter. Along with this the dataset contains the category of the disease, the actual disease and the symptoms related to this disease. We used SNOMED-CT for obtaining the relation between the category and the diseases. For the purpose of visualization we assume that the patient encounters a number of diseases throughout his/her record span.

Patient 1 is a 94-year-old male who had 1427 medical records between the years 1980 to 1989.From the violin plot, we can see that very frequently she had medical conditions related to Cardiology and to understand what diseases were observed we can select Cardiology and it can be seen what diseases related to Cardiology were observed and the plot tell us that patient 1 had Congenital heart disease very frequently and this information can be used to create treatment plans since it’s very frequent one, then they also can check what treatment was done and modify it since the previous plan is not helping. To understand deeper about what symptoms caused a particular disease you can select on any and it opens a view with different symptoms and it’s frequency the same logic can be used to diagnose those symptoms and helps in creating a new treatment plan if needed.

Patient 2 is an 83-year-old female who had 1427 medical records between the years 1980 to 1989. From the violin plot, we can see that very frequently she had medical conditions related to Nephrology and Endocrinology and to understand what diseases were observed we can select Nephrology and it can be seen what diseases related to Nephrology are observed and also how frequently they are observed. This information can be used to create treatment plans in situations like if the disease she got now is a very frequent one, then they also can check what treatment was done and modify it since the previous plan is not helping. To understand deeper about what symptoms caused a particular disease you can select on any and it opens a view with different symptoms and it’s frequency the same logic can be used to diagnose those symptoms and helps in creating a new treatment plan if needed.

## 4. **Discussion**

The original paper contains visualizations of the frequencies of diseases and the symptoms obtained from SNOMED-CT. To visualize and analyze the data at a deeper level with respect to relations between the categories, diseases and symptoms we have created a tree visualization as an extension.

Further if the allergy data of patients is obtained then these allergies can be plotted as a bar graph by taking into account the frequency of these records over various time periods which is extracted from the dataset. We think that creating such visualization will be helpful not only for patients to keep track of their allergies but also for the doctors to prescribe medications to patients in the future.

In addition to the above observations we found that if the data contains information about the hospital transfers, for example, if a patient was transferred from the general ward to ICU then it might be useful for extracting more information through visualization. We propose a visualization for analyzing this data and extracting pattern for the degree of severeness that was recorded for that patient by possibly using a line chart. We think that by doing this the overall health of the patient can easily be visualized in a single graph and will be potentially helpful for doctors to not only prescribe medications with the specific strengths and doses of medicines but also will be helpful for faster response with respect to transfers and admissions to hospitals.

## **ACKNOWLEDGMENTS**

We would like to thank Prof.Chris Bryan for his contribution during our proposal and also for teaching us the key concepts in Data Visualizations which we used in our project.

## **REFERENCES**

\[1\] Sultanum, Nicole, Devin Singh, Michael Brudno, and Fanny Chevalier. "Doccurate: A Curation-Based Approach for Clinical Text Visualization." *IEEE transactions on visualization and computer graphics* 25, no. 1 (2018): 142-151.

### Github link:
https://github.com/asu-cse578-f2019/Doccurate-Piyush-Kshitij-Pranjal-Aniket-Sai/

