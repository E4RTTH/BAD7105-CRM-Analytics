# Customer Segmentation
[![](https://img.shields.io/badge/-Python-blue)](#)

**Data Set** Supermarket Data

**Note book** [Customer Segmentation](./Customer_Segmentation_Group_10.ipynb)

## Method

### 1. Prepare customer single view
### 2. Cluster customers
### 3. Compare model performance
</br>

![image](./model_performance.png)

### 4. Kmeans Clustering (5 clusters)
</br>
    
![image](./Cluster.png)

![image](./Elbow.png)

### 5. Interpret results and plan for actions
</br>

![image](./cluster_interpret.png)

![image](./Interpret1.png)

![image](./Interpret2.png)

## Result


<span style="color: #85C1E9 ">**Cluster 0 : Slots**


*  Most Mean time between purchase
*  Least ticket size
*  Frequent visit on monday

#### Action: Ignore them
---


<span style="color: #85C1E9 ">**Cluster 1 : Unicorn**


*   Most spend
*   Most buy various of SKUs
*   Most ticket size
*   Most S.D. spending
*   Most spend fresh
*   Most spend store region e02

#### Action: Privilege
---


<span style="color: #85C1E9 ">**Cluster 2 : Donkey**


*   Least spend
*   Least visit
*   Most recency
*   Least S.D. ticket size

#### Action: Upsell and Discount
---


<span style="color: #85C1E9 ">**Cluster 3 : Pegasus**


*   Most visit
*   Least recency
*   Least Mean time between purchase
*   Have most relationship time with store
*   High S.D. spending
*   Most spend grocery
*   Most spend store region w01

#### Action: Promotion
---




<span style="color: #85C1E9 ">**Cluster 4 : Horse**


*   Middle S.D. spending
*   Middle spend
*   Middle visit

#### Action: Upsell and Discount


