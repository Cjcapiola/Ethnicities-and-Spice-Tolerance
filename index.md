# How does one’s ethnic descent (ancestry) affect tolerance to spicy foods?
## Introduction
After a conversation between Raphael Kalatzis and I, it was determined that a sufficently motivated and interesting data science question would involve determining if there was any correlation between spice tolerance and ethnicity. This specific negotations related to how this question should be best answered is detailed in the disscusion section below. This analysis' main constraint is the avaliability of data so a creative solution to answering this problem must be generated and supported. The use of LLM will aid in the data collection and analysis of this project, while the thoughts, disscusion, and conclusions based on the data will be informed by ongoing conversation between Raphael and myself. By the end of this post, the reader will be able to decern whether there is a relationship between ethnicity and spice toelrance and to what degree is this effect observed. The reader will also be able to see creative principles of data science in action, the importantce of ongoing communication and collaboration, as well as the use and limitations of LLMs in a data science project.


## Methods

There is limited avaliability of data from direct experimental analysis which tested spice tolerance across a sufficently large sample size to draw any meaningful conclusions. However, there was one informative study found called *"Why do some like it hot? Genetic and environmental contributions to the pleasantness of oral pungency"* by Outi Törnwall et. al. [^1] which assesed the various factors the contibuted to spice preference. The assumption made in this data analysis, which is supported from this paper, is that tolerance and preference are strongly correlated such that we can say that those who have a preference for spice foods likely have a greater tolerance for spice than those who do not enjoy spicy foods. This paper also provides some basic grounds for this analysis as it concluded that genetic factors accounted for 18-58% of the variation in the pleasantness spicy foods. Therefore this analysis aims to specifically correlate spice preference with ethnicity to answer the main investigative question.

As mentioned, there is a lack of avaliability in data sets that would enable us to draw specific and direct conclusions. However, we can indirectly related preference and ethnicity by analysing the top 300 "Best Ethnic Cookbooks" as ranked by users of Goodreads. The link to this list can be found [here.](https://www.goodreads.com/list/show/1922.Best_Ethnic_Cookbooks?page=1) The collection and processing of data is detailed step by step below:

1. The top 300 "Ethnic Cookbooks" were manually pulled from the Goodreads list and inserted into an excell file that contained the ranking, title, and author of the book.
2. The datasheet underwent subsequent processing to include only the titles of the cookbooks. A list of standardized racial and ethnic classifcations was obtained from Harvard Univserity which basis its classifcation system to be in accordance with the standards mandated by the U.S. Department of Education. That list can be found [here.](https://hr.harvard.edu/files/humanresources/files/race_ethincity_definitions_2014.pdf) A LLM was used to scrape the titles for any relevant keywords that correlated the titles with a specific ethnicity, region of the world, or specific culture and then was asked to match those identifiers to the standardized list provided by Harvard University.
3. The list of 300 "Ethnic Cookbooks" was trimmed down to 106 based off the inclusion criteria stated above. If there was not sufficent information in the title to connect it with one of the standard ethnic classifcations it was removed from the analysis. The orginigal list can be found [here](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/blob/main/cookbooks.xlsx) and the cleaned, sorted, and classified list can be found [here.](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/blob/main/cleaned_ethnic_cookbooks.xlsx) The code that the LLM produced to run this process is detailed below:
```python
   import pandas as pd

# Load the provided Excel file
file_path = '/mnt/data/cookbooks.xlsx'
cookbooks_df = pd.read_excel(file_path)

# Cleaning the DataFrame to keep only the book titles
# Filtering out rows that contain NaN in the third column (which should represent titles)
cleaned_cookbooks_df = cookbooks_df[cookbooks_df.iloc[:, 2].notna()].reset_index(drop=True)

# Keeping only the column with the titles
cleaned_cookbooks_df = cleaned_cookbooks_df.iloc[:, 2]

# Renaming the column for clarity
cleaned_cookbooks_df = cleaned_cookbooks_df.to_frame(name='Title')

# Further cleaning to remove rows that don't contain actual book titles
# Define keywords that indicate a row is not a book title
non_title_keywords = ['avg rating', 'score:', 'by\xa0', 'people voted']

# Filter out rows containing these keywords
cleaned_cookbooks_df = cleaned_cookbooks_df[~cleaned_cookbooks_df['Title'].str.contains('|'.join(non_title_keywords))]

# Resetting the index
cleaned_cookbooks_df = cleaned_cookbooks_df.reset_index(drop=True)

# Mapping function to identify the ethnicity based on the book title
def map_ethnicity(title):
    # Keywords for different ethnicities based on the provided list
    ethnicity_keywords = {
        'White (not of Hispanic origin)': ['European', 'Italian', 'French', 'German', 'Greek', 'Russian', 'Middle East', 'North Africa'],
        'Black (not of Hispanic origin)': ['African', 'Caribbean'],
        'Hispanic or Latino': ['Mexican', 'Puerto Rican', 'Spanish', 'Cuban', 'South American', 'Central American'],
        'Asian': ['Chinese', 'Indian', 'Japanese', 'Korean', 'Thai', 'Vietnamese', 'Pakistani', 'Southeast Asian', 'Far East'],
        'American Indian or Alaska Native': ['Native American', 'Alaskan', 'Navajo', 'Cherokee'],
        'Native Hawaiian or Pacific Islander': ['Hawaiian', 'Guam', 'Samoan', 'Pacific Island']
    }

    # Iterate over the ethnicities and their keywords
    for ethnicity, keywords in ethnicity_keywords.items():
        for keyword in keywords:
            if keyword.lower() in title.lower():
                return ethnicity
    return None

# Apply the mapping function to the DataFrame
cleaned_cookbooks_df['Ethnicity'] = cleaned_cookbooks_df['Title'].apply(map_ethnicity)

# Filter out rows where ethnicity couldn't be identified
filtered_cookbooks_df = cleaned_cookbooks_df.dropna(subset=['Ethnicity'])

# Save the cleaned and updated DataFrame to a new Excel file
excel_save_path = '/mnt/data/cleaned_ethnic_cookbooks.xlsx'
filtered_cookbooks_df.to_excel(excel_save_path, index=False)
```
5.  Now that the list of eligible cookbooks has been collated, the titles were manually searched using Google to find book recipie exerpts. These excerpts were then searched for ingredients and the ingredients list was inserted alongside the title of the book in the excel document.
6.  The ingredients list was then analyzed for the relative proportion of "spicy ingredients". This was done by analysing the ingredients list of the excerpts provided and extracting the spices and flavorings used. Once this was done, the specific spices used were evaluated for their capsasin content via a simple web search. The average capsasin content for each given book was then found and listed. Now we have an excel document that contains the cookbook title, the assigned ethnic classification, and the average capsasin content of the listed ingredients. From here we can continue with a data analysis that attempts to find a correlation between the average capsasin levels and a given ethnicity. The document can be found [here](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/blob/main/updated_cookbooks_with_spice_scores.xlsx) and the process of data analysis is detailed in the following section.


## Data Analysis
Now that the data has been collected we can proceed with a throughouh analysis to uncover any trends found between the spice score and ethnic classifcation. In order to provide a comprehensive analysis we will:
1. Perform a Statistical Summary: Calculate the mean, median, and standard deviation of spice scores for each ethnic group.
2. Ethnicity-based Comparison: Use an ANOVA and Tukey-Post Hoc test to compare the spice scores across different ethnicities to see if certain ethnicities tend to have higher or lower spice scores and if they are significant.
3. Do a more in depth Ingredient Analysis: See if the presence of certain spices like chili peppers or cayenne pepper consistently correlates with higher spice scores.
4. Visualize the data to show any found relationships in an easy to interpret way.
5. Draw conclusions.

### Summary Statistics
* Asian Cuisine:
   * Number of Books: 55
   * Mean Spice Score: 8.74
   * Standard Deviation: 6.27
   * Minimum Score: 0.20, Maximum Score: 24.91
* Black (Not of Hispanic Origin) Cuisine:
   * Number of Books: 9
   * Mean Spice Score: 7.97
   * Standard Deviation: 5.74
   * Minimum Score: 0.50, Maximum Score: 15.30
* Hispanic or Latino Cuisine:
   * Number of Books: 8
   * Mean Spice Score: 4.77
   * Standard Deviation: 3.30
   * Minimum Score: 0.80, Maximum Score: 11.35
* White (Not of Hispanic Origin) Cuisine:
   * Number of Books: 34
   * Mean Spice Score: 9.77
   * Standard Deviation: 6.22
   * Minimum Score: 0.04, Maximum Score: 21.80

Asian and White cuisines seem to have a higher range and median of spice scores compared to Black and Hispanic/Latino cuisines. However, it's important to note the smaller sample size for Black and Hispanic/Latino cuisines, which might affect the representativeness of these results.
The variability within each ethnic category, as shown by the standard deviation, suggests a diverse range of spice levels even within the same ethnic cuisine.

### ANOVA and Tukey-Post Hoc
The spice scores and ethnicites were subject to a ANOVA and Tukey Post-Hoc test using MATLAB. The code used to generate this test is shown below:
```matlab
function conductANOVA()

% Locate and load Excel data file
[file,path] = uigetfile('*.xlsx','Select Excel Data File');
dataSet = readtable(fullfile(path,file));

% Conduct one-way ANOVA
ydata = dataSet{:,1};
group = dataSet{:,2};
[p,anovaResults,stats] = anova1(ydata,group,'off');

% Convert anovaResults to data table to be displayed later
anovaData = anovaResults(2:end,2:end);
anovaTable = cell2table(anovaData);
AnovaRowNames = anovaResults(2:end,1);
AnovaColumnNames = anovaResults(1,2:end);
anovaTable.Properties.VariableNames = AnovaColumnNames;
anovaTable.Properties.RowNames = AnovaRowNames;

% Tukey multiple comparisons
[c,~,~,gnames] = multcompare(stats,'display','off');
tbl = array2table([c(:,1),c(:,2),c(:,6)],...
	'VariableNames',{'Group_vs','Group','p-value'});
tbl.("Group_vs") = gnames(tbl.("Group_vs"));
tbl.("Group") = gnames(tbl.("Group"));

% Display ANOVA and Tukey comparison tables
fig = uifigure('Name','ANOVA Results','NumberTitle','off');
fig.Position(3:4) = [550 320];
uitable(fig,'Data',anovaTable,'Position',[20 165 500 120]);
uicontrol('Parent',fig,'Style','text','Position', [20 285 200 20],...
    'String', 'One-way ANOVA Table', ...
    'HorizontalAlignment','Left');

uitable(fig,'Data',tbl,'Position',[20 20 500 120]);
uicontrol('Parent',fig,'Style','text','Position', [20 140 200 20],...
    'String', 'Tukey Multiple Comparisons', ...
    'HorizontalAlignment','Left');

figure('Name','Box Plots','NumberTitle','off');
boxplot(ydata,{group},...
    'ColorGroup',group);
xlabel('Group');

end
```

The results of the test are shown below:

![image](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/assets/143909827/933f6f06-5755-4d9a-90a1-50c85a8cc974)


![image](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/assets/143909827/705ec86c-d225-4caf-bafc-7019378b58d9)

As can be seen from the test, there was no significant difference found in any of the groups with an overall p value at 0.2125 which would not be signficant when considering an alpha of 0.05. Despite this conclusion, it is interesting to note the deviations present in each of the ethnic categories. There is also a substantial outlier present in the Asian Ethnic Category which signnifies that there is one particular cookbook with an Asian designation that contains especially spicy recipies. 

## Results


## Disscussion
# Commentary on Results

# Limitations of Study

# Iterative Conversation between Raphael Kalatzis and I

# Limitations of CHAT-GPT as a LLM for Data Analysis

## Conclusion




[1^]: Törnwall O, Silventoinen K, Kaprio J, Tuorila H. Why do some like it hot? Genetic and environmental contributions to the pleasantness of oral pungency. Physiol Behav. 2012 Oct 10;107(3):381-9. doi: 10.1016/j.physbeh.2012.09.010. Epub 2012 Sep 23. PMID: 23010089.
