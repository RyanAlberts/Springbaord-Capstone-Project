July 15, 2017


Springboard
New York, NY
 
	Capstone Project Milestone Report
 
	Introduction: 
Walmart’s acquisition of Jet.com for $1.3 billion and Amazon’s acquisition of Whole Foods for $13.7 billion are symptomatic of an anxiety surrounding the future of retail. No superlatives have been spared to describe the trend, despite eCommerce accounting for no more than 9% of US retail spend, as of Q1 2017 (according to the Census Bureau). At its very core, this shift falls under the domain of Supply Chain Management; products and services are experiencing a wildly different lifecycle between the point of production and the point of consumption.  
	The result is that retailers with lean, adaptive supply chains will win. The implicit imperative behind retail’s changing landscape is the need to predict and adapt to consumer habits, which makes for some supremely interesting Supply Chain questions. 

        Instacart Kaggle Competition: 
Goal: Predict the products that will be in each user’s next order. 
Evaluation Metric: Mean F1 Score
Data: Over 3M orders from 206k customers

Instacart is an online grocery and delivery app. Instacart’s shopping experience is driven by data scientists that curate the products that you see. The current $20 billion Americans spend at online groceries represents a meager 2.5% of the estimated $800 billion US grocery market, where perishable goods make predicting consumer buying habits a matter of life or death (figuratively, except for the produce). Demand Forecasting being a cornerstone of Supply Chain Management (SCM), I see this challenge as an opportunity to explore the domains that have driven my career thus far – eCommerce and SCM – through the lens of data science. 

Please consult the competition website for more details:
https://www.kaggle.com/c/instacart-market-basket-analysis

	Client:

Instacart. 
	
	Preliminary Exploration & Initial Findings:

I initially explored the data by importing the six (6) data files into a number of Jupyter iPython notebooks, which I organized by theme. The environment provided an ideal interface for convenient, clean, and visually effective exploration, as well as providing a robust story-telling framework via LaTeX and piecemeal code execution. The data is relatively clean and well-wrangled, which presented an ideal opportunity to focus on analyzing patterns that I would later build into features for my predictive algorithm. 
	I familiarized myself with the data within the context of the goal of the Kaggle competition, which is to predict what users will order next. The data reveals that there are several key insights necessary to produce such a predictive algorithm. The basic assumptions of such predictive algorithms are covered well in the Springboard coursework as well as in a few papers sited at the end of this report. In the context of this competition, the most basic assumption is that past order history has some bearing on future orders, including the actual products that a user buys (assortment) and the size of their order (basket size), among other things. It also assumes a user’s habits, including order timing, frequency, product loyalty, etc., are imperfect but capable indicators that influence the probabilities of ordering specific products in the future. It may also be shown that larger aspects of the shopping experience influence macro-level trends that are customer-agnostic in the sense that they can have measurable impact on large groups of customers. For example, 24 out of the 25 most popular products late at night are ice-cream. 
	While I used the Python language as a basis for driving my exploration, I relied heavily on numpy for linear algebra and numerical manipulation, pandas for data processing and frameworks, as well as matplotlib and seaborn for visualization. I also made extensive use of several built-in libraries in Python, including os for file-handling, re for regular expressions, datetime and calendar for various time-series manipulations, and basic Unix from the command line to setup and run my coding environment. I’ll address machine-learning algorithms later.
I created three notebooks with my initial findings, listed below. The notebooks provide more of a narrative than I will present here, so please consult the links for more in-depth exploration.


1.	Categories – General
a.	This file explores products and product categories, including relationships between what Instacart calls ‘departments’ and ‘aisles’, which correspond roughly to ‘category’ and ‘sub-category’ respectively. 
i.	https://github.com/RyanAlberts/Springbaord-Capstone-Project/blob/master/Instacart_EDA_Categories__General.ipynb
2.	Assortment
a.	This notebook dives a deeper into how a user’s product assortment (the particular combination of products they include in each order) affects reorder behavior. I attempted to explore how category affinities, basket sizes, product diversity, and order frequency  interact to influence user behavior. 
i.	 https://github.com/RyanAlberts/Springbaord-Capstone-Project/blob/master/Instacart_EDA_DataStory_Assortment.ipynb
3.	Engagement
a.	This notebook incorporates the previous two and served as a launching pad for feature engineering. The notebook explores how user engagement of various sorts affects reorder behavior. It attempts to address issues like ‘New’ vs. ‘Seasoned’ customer order behavior, basket sizes and order frequency, and order cyclicality.
i.	https://github.com/RyanAlberts/Springbaord-Capstone-Project/blob/master/Instacart_EDA_DataStory_Engagement.ipynb

	Deep Dive – Exploring the Data: 
	
I did a deep-dive into user_id 1 in the Engagement notebook, which served a s the basis for the ‘EDA_FeatureSet’ Notebook. The data revealed some interesting trends. For any given customer, there is often a long-tail of products that are never reordered, and a small but formidable group of products that are consistently reordered. There was also a strong correlation between the ‘days_since_prior_order’ variable and the basket size and assortment of any particular order. The ‘EDA_FeatureSet’ Notebook provides the basis for a lot of feature engineering that I later developed for the machine-learning algorithms, and provided a cauldron for improving my skills with grouping, merging, indexing, and applying methods to various pd.DataFrame and Series objects. 
Given the size of the data, it was an ideal way of learning first-hand how crucial lean, efficient, modular programming is, including list comprehensions, apply methods, and other sorting and grouping techniques. I’ll expand on that with examples in the capstone write-up, but suffice it for now to say that these techniques were essential to exploring the data, considering you can easily loose track of a line of thought when it takes your program hours to iterate through a for loop, or to iterate through a dataset where the dtypes haven’t been optimized for the purpose. 
	Despite it becoming clear, from the Kaggle discussion forums and my own experience, that there is no ‘magic’ feature that is capable of predicting reorder behavior across the 3M orders, there are some general trends that are worth noting. For example, the more recent order history (e.g. last 4 orders) are more capable of predicting future order behavior than an averaging of the entire order history of a customer (maxing out at 100 orders). Also, employing statistical analyses of the central tendencies of customer behaviors (e.g. the coefficient of variability for the basket size, or the standard deviation of the number of orders since that user bought a specific item) proved remarkably capable of establishing confidence in the probabilities associated with reordering certain products. For example, we can be more certain of the probabilities associated with certain products for a customer that shows very little variance in the time and regularity of their order (i.e. regularly ordered at 9AM on Monday morning every week) as compared to someone who orders erratically and infrequently. Thus we can weight the probabilities associated with the first user more heavily than those associated with the second user. 
	However, I saw that the data is highly unbalanced. With three million orders and 4.8M (user_id, product_id) pairs in the test set alone, there is a high degree of variability regarding reorder behavior, which made it clear that over-fitting would be a major issue in crafting a model that learns from an unbalanced training set. 
	
        Initial Model: Unrefined, but Capable 

I built my initial predictive model without any sort of machine-learning library, as a way of benchmarking my progress. At its core, my initial model was built to look at every single product a user ordered, and reduce that set to a smaller sub-group of products that were likely to be reordered. I created a relatively robust, albeit unsophisticated, mapping of input (user-product pairs) to output (user-product pairs) based on several static conditional operations that identified products to include and products to exclude based on meeting various criteria. For example, if the product was new in the most recent order, the product would be excluded, and if the product had been ordered in two or more of any of the previous 5 orders, it would be included. 
This initial model was rough. It scored as high as .365, which was enough to get me to the top 50%, but the amount of time and energy to progress became exponential at a point, because criteria for inclusion or exclusion were basically independent (meaning refining one might detract from another, and the net loss in accuracy increased with each new criteria, due to increasing complexity), were uniform across users (meaning, unlike the data, the criteria did not vary based on user, despite the obvious fact that user behavior is idiosyncratic), and did not take into account things like basket size and statistical measures of variance. It was a great benchmark, but not enough to get to the top of the ranks. 
That said, I found the benchmark models extremely valuable because I was able to interact with the training and test data sets and see how certain categories of criteria are more important than others (e.g. recent order history is hugely important, but whether a product is new in the most recent order is less useful, for predictive purposes). These trends helped me immensely when it came time to employ machine-learning algorithms. 

	Current Priorities and Next Steps:
I am currently in the process of deploying a few Gradient Boosting Machines (GBMs) and Random Forests algorithms. The bulk of this work has yet to be completed, and I look forward to reporting their efficacy and my progress in the Capstone Report Write-Up. 
I expect the bulk of my time to be spent between feature engineering for the ML algorithms, parameter-tuning, threshold-setting for binary classifier GBM, and building in methods of addressing fringe cases, like predicting whether or not a user will not reorder anything that they’ve ordered before (‘None’-handling). I am learning a great deal from the Kaggle discussion boards as far as building in F1 optimization code into the predictive models, and will report my conclusions in the write-up.

	Progress Evaluation:

My ultimate goal is to use this experience, including immersing myself in the Kaggle discussions and culture of collaborative open-source analysis, to be able to function at a high level within an organization both from an Operations & Strategy perspective, as well as from a Data Science perspective, focusing on applied strategy and decision-making. I have learned a great deal regarding machine-learning programs, statistical applications in data science, and how to consume and adapt public code repositories for the purpose at hand, as well as iterate off of collaborative coding environments in a productive and competitive manner. I look forward to having a production-ready predictive model capable of handling intense Demand-Forecasting applications for Instacart, and the skills to transfer these capabilities to future endeavors.
