# How does one’s ethnic descent (ancestry) affect tolerance to spicy foods?

## Introduction
After a discussion with Raphael Kalatzis, we determined that an intriguing data science question would be to explore any correlation between spice tolerance and ethnicity. The negotiation of how to approach and answer this question is detailed in the discussion section below. The main constraint of this analysis is the availability of data, necessitating a creative approach. The use of an LLM will aid in data collection and analysis for this project, with insights, discussions, and conclusions drawn from ongoing dialogue between Raphael and myself. By the end of this post, readers should discern whether a relationship exists between ethnicity and spice tolerance and to what extent this effect is observed. This post also demonstrates creative data science principles, the importance of ongoing communication and collaboration, and the capabilities and limitations of LLMs in a data science project.

## Methods

Data from direct experimental analysis testing spice tolerance across a sufficiently large sample size is limited. However, one informative study titled "Why do some like it hot? Genetic and environmental contributions to the pleasantness of oral pungency" by Outi Törnwall et al[^1] assessed factors contributing to spice preference. This analysis assumes a strong correlation between tolerance and preference, suggesting that those who enjoy spicy foods likely have a higher tolerance. This paper also establishes a foundation for our analysis, concluding that genetic factors account for 18-58% of the variation in the pleasantness of spicy foods. Our analysis aims to correlate spice preference with ethnicity.

Given the lack of directly relevant datasets, we approached the question by analyzing the top 300 "Best Ethnic Cookbooks" as ranked by Goodreads users. The list can be found [here](https://www.goodreads.com/list/show/1922.Best_Ethnic_Cookbooks?page=1). The data collection and processing steps are as follows:

1. Manually compile the top 300 "Ethnic Cookbooks" from Goodreads into an Excel file with the ranking, title, and author.
2. Process the datasheet to include only the titles. Obtain a list of standardized racial and ethnic classifications from Harvard University, which complies with U.S. Department of Education standards, found [here](https://hr.harvard.edu/files/humanresources/files/race_ethnicity_definitions_2014.pdf). Use an LLM to match title keywords with ethnic identifiers from the Harvard list.
3. Narrow down the list to 106 cookbooks based on the inclusion criteria. Titles that did not provide sufficient information for ethnic classification were removed. The original list is available [here](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/blob/main/cookbooks.xlsx), and the cleaned list is [here](https://github.com/Cjcapiola/Ethnicities-and-Spice-Tolerance/blob/main/cleaned_ethnic_cookbooks.xlsx).

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

### Ingredient Analysis

It was found that the most common ingredients used overall were Onions, Nutmeg, Garlic, Basil, and Rice with  Onions and Nutmeg being the most frequent, appearing in 23 cookbooks each. When analyzed it was found that high spice score groups contained disproportionate amounts of Garlic, Nutmeg, Paprika, and Chilie while low spice score groups contained more sweet ingredients and starches such as Rice, Sweet Potato, Sugar, Cinnaimon, and Salt. The correlations, while insightful, are not strong enough to decisively determine spice levels. This suggests that spice levels in recipes are influenced by a combination of various ingredients and not just a few specific spices. With specific consideration to specific cusine types it was found that:
* Asian Cuisine was characterized by ingredients like Ginger, Soy Sauce, Coconut Milk, which indicate a preference for moderately spicy flavors, often balanced with sweet or tangy elements. Combinations like Ginger + Soy Sauce or Coconut Milk + Curry Leaves are frequent in higher spice score dishes.
* Hispanic or Latino Cuisine was characterized by ingredients like Cilantro, Lime, Avocado with lower spice scores suggestesting a focus on fresh, vibrant flavors rather than intense heat. This was a suprising finding and suggests that some of the cookbooks in the list, which was created by popular rating of Goodreads users, might have a preference for more "Americanized" Hispanic food where spice is toned down. Combos like Cilantro + Lime or Beans + Avocado are common in dishes with lower spice scores.
* White (not of Hispanic origin) Cuisine saw ingredients like Basil, Oregano, Olive Oil with an emphasis on  the use of herbs and aromatic spices rather than heat-inducing ingredients directly. Pairings like Basil + Tomato Paste or Garlic + Olive Oil were found emphasiing the influence of Mediteranian Cusine on the overal data set.


## Results

### Statistical Analysis of Spice Tolerance Across Ethnicities
We conducted a comprehensive statistical examination to explore variations in spice tolerance across different ethnic cuisines, as represented by spice scores in cookbooks. Our dataset included cookbooks categorized under four ethnic groups: White (not of Hispanic origin), Black (not of Hispanic origin), Hispanic or Latino, and Asian.

### ANOVA and Post-hoc Analysis
An ANOVA test was performed to determine whether there were significant differences in the mean spice scores across the aforementioned ethnic groups. The test yielded an F-value of 1.536, with a P-value of 0.2125, suggesting no statistically significant differences in spice scores between the groups. The lack of significance was further corroborated by Tukey's HSD post-hoc analysis, where all pairwise comparisons between ethnic groups had P-values above the conventional threshold of 0.05.

### Distribution and Variability of Spice Scores
Despite the lack of statistical significance in mean differences, we visualized the distribution of spice scores using box plots. These plots revealed varied medians and ranges within each ethnic category. The Asian group showed the widest range of spice scores, while the Hispanic or Latino group exhibited the narrowest range.

### Correlation Between Ingredients and Spice Scores
Our correlation analysis delved into the relationship between specific ingredients and spice scores. It indicated that certain ingredients, such as Hoisin Sauce, Ginger, and Cinnamon, exhibited a positive correlation with higher spice scores. Conversely, ingredients like Onions and Seaweed showed a negative correlation, being more prevalent in cookbooks with lower spice scores.

### Frequency of Ingredients
The frequency analysis of ingredients within the cookbooks unveiled that Onions, Nutmeg, and Garlic were the most common across all ethnic groups. However, the presence of these ingredients did not significantly influence the spice scores, suggesting that they are foundational rather than indicative of spice levels.

## Discussion

### Commentary on Results
The findings from our data analysis revealed intriguing patterns in spice tolerance across different ethnic cuisines, as represented by spice scores in cookbooks. Although the statistical analysis did not indicate significant differences in spice tolerance among the ethnic groups examined, the absence of statistical significance does not diminish the cultural and gastronomic value these spices represent within their respective cuisines. The correlation analysis highlighted ingredients that potentially influence spice scores, with items like Hoisin Sauce, Ginger, and Cinnamon showing a positive relationship with higher spice scores. This suggests a complex interplay between spices and perceived spiciness in dishes.

Interestingly, staple ingredients such as Onions and Nutmeg were ubiquitous across all ethnicities, emphasizing their foundational role rather than a direct contribution to spiciness. This ubiquity points to a shared gastronomic thread among diverse culinary traditions. The broad range of spice levels within each ethnic category underscores the richness of culinary practices and highlights that spice tolerance and preference are influenced by a tapestry of factors beyond ethnicity alone.

### Limitations of Study
The primary limitation of this study is the indirect approach used to assess spice tolerance. Employing cookbook spice scores as a proxy for individual spice tolerance introduces a level of abstraction that may not accurately capture the personal sensory experience. Furthermore, the sample size for certain ethnic cuisines was relatively small, which could impact the generalizability of the findings.

Another limitation was the reliance on the interpretation of cookbook titles to infer ethnicity, which may not always reflect the complexity and fusion inherent in contemporary culinary practices. Future research would benefit from direct sensory testing across diverse populations to more accurately measure spice tolerance and its relationship to ethnicity.

### Iterative Conversation between Raphael Kalatzis and I
At the outset of this analysis, Raphael initially framed his question in a manner best addressed through a research study protocol. He suggested selecting subjects from diverse ancestries and subjecting them to DNA tests from ancestry.com or 23andMe. He then proposed surveying these subjects about their spice tolerance and subjecting them to the Scoville Heat Scale Challenge and a Capsaicin Sensitivity Test. Although these methods would be valuable in an experimental context, they were impractical for the scope of this project.

We then explored various methods that could indirectly answer the posed question. This included collating information related to different ethnic restaurants and analyzing the reviews left at these establishments, which could then be ranked based on reported spice levels on the menu. Positive reviews of these restaurants would indicate that the reviewer had a high spice tolerance, and the ethnicity of the reviewer could potentially be analyzed to develop a correlation. This method presented substantial challenges in data availability and the privacy and political concerns of subjectively categorizing reviewers into ethnic categories. Other approaches considered included analyzing the frequency and context of spicy ingredients mentioned in online recipe databases and culinary forums, examining the sales data of spicy products, and leveraging existing nutrition and dietary studies that include ethnic background data. However, the need for sophisticated data processing tools and the requirement to navigate issues related to privacy and data use ethics presented too great a hurdle in this timeframe.

Raphael expressed disappointment at the lack of a significant result from the analysis but was generally satisfied with the analytical methods and indirect measurements used. Even without a significant relationship found, Raphael felt that interesting results could still be gleaned. He jokingly planned to tease some of his friends for their lack of spice tolerance despite belonging to a group correlated with higher spice level recipes.

One critical comment made was regarding the classification scheme based on the Harvard University scale. Raphael felt that the bins provided by this scale were too broad, resulting in a generalization of cultures. He cited his Greek descent and purported lack of spice tolerance compared to some of his Middle Eastern friends who could eat very spicy foods. In this analysis, these two groups would be lumped into the same "White (not of Hispanic descent)" category, despite significant differences. Future investigations should develop more nuanced categorization techniques that are more specific than ethnicity. Additionally, while this project focused on a genetic basis for spice tolerance, other investigations should consider the assured effect of culture and home influence on spice tolerance.


### Limitations of CHAT-GPT as a LLM for Data Analysis
While CHAT-GPT has useful in assisting with data analysis and interpretation, it has inherent limitations. As a language model, CHAT-GPT's analysis is confined to the data provided and lacks the ability to perform independent data collection or verification. Its interpretations, based on programmed algorithms, cannot replace the nuanced understanding a human analyst brings, especially in the context of cultural and subjective topics like food and taste.

Additionally, CHAT-GPT's outputs are contingent on the quality of the instructions and data it receives. Any biases or errors in the input data can lead to skewed results. The model's lack of real-world awareness or the ability to experience taste means its analysis must be supplemented with human insight. Ethical considerations around privacy and consent in data collection, as well as adherence to OpenAI's use-case policy, further limit its research applicability.

In this project, the LLM required direct manual interpretation and data entry, significantly affecting the feasibility of a project using these principles at scale. Until APIs are developed that allow CHAT-GPT to scrape data from scientific literature on capsaicin content in foods and enable the LLM to directly analyze cookbook contents, the scope of this methodology will be limited.


## Conclusion

Our analysis suggests that while observable differences in spice scores across ethnic cuisines exist, they are not statistically significant. The ingredient analysis provided insights into the complex interplay between various components that contribute to the spice profiles of ethnic cuisines. The results highlight the rich diversity and multifaceted nature of spice usage, which transcends simple categorizations based on ethnicity.




[1^]: Törnwall O, Silventoinen K, Kaprio J, Tuorila H. Why do some like it hot? Genetic and environmental contributions to the pleasantness of oral pungency. Physiol Behav. 2012 Oct 10;107(3):381-9. doi: 10.1016/j.physbeh.2012.09.010. Epub 2012 Sep 23. PMID: 23010089.
