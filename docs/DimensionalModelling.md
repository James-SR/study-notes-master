# Dimensional Modelling
***

## Introduction to Dimensional Data

Dimensional modelling helps to build the ability for users to query the information, for instance analysing results by a geographic region.  Multi-dimensional modelling is an extension to allowing multiple ways to analsye the information, by geographic region but also over time, by product or service, by store or office and so on. It provides a way for a system user, manager or analyst to navigate what information - the 'information space' - is available in a database or data warehouse, but at a more intuitive level [see @FIT5195, lecture 4].  The goal is to help understanding, exploration and to make better decisions.   A dimension is simply a direction, usually query or analytically based, in which you can move.  

The dimensions used therefore become the ways in which the end user wants to query the information.

Typical terms used in the BI arena for helping to navigate this 'information space' include; 'slice and dice' meaning to make a large data space in to a smaller one (you are making a selection or subset of all the available data), 'drill down' meaning to go in to a lower level of a hierachy (moving from a geographic region to a particular store), 'drill up' meaning to go in to a higher level (sometimes called rolling-up) and 'drill across' meaning adding more data (or facts) about something, typically from another source (a different fact table).

There are three aspects of information with a Business Intelligence system - conceptual, logical and physical - which exist on a spectrum.  

* Conceptual - The business needs are usually the high level conceptual solution, what things we want to include at a more abstract level
* Logical - We start thinking about what data to include in the model and what data is available, it starts giving something which can be implemented in to a warehouse
* Physical - The final solution which is usually then what is implemented in the data warehouse. It is the more technical/IT solution and may include normalisation (3NF or higher) and perhaps other database optimisations to improve performance of the system.

In some instances, the conceptual and logical can become one and the same thing.    

Table: (\#tab:simple-table) The three levels of data modelling

Feature                Conceptual    Logical    Logical   
--------              ------------  ---------  ---------
Entity Names	             Y            Y
Entity Relationships	     Y            Y
Attributes	 	                          Y          
Primary Keys	 	                        Y          Y
Foreign Keys	 	                        Y          Y 
Table Names	 	 	                                   Y
Column Names	 	 	                                 Y
Column Data Types                                  Y

## Kimball Approach

Before work begins of the data modelling, it is neccessary to understand the needs of the business and the underlying data [@Kimball2013, pg 37].  The business needs arise out of meetings with manangers, decision makers and other representatives of the business.  Kimball also recommends meetings with _'source system experts and doing high-level
data profiling to assess data feasibilities'_ [@Kimball2013, pg 38].  Whilst the data modeller is 'in charge' the actual model should unfold via a series of interactive workshops with those business representatives.  Data governance reps should also be involved to obtain buy-in.  In this sense, the Kimball approach covers both the conceptual and physical, it may also include some considerations of physical level at initiation.

## Four-Step Dimensional Design Process

Kimball outlines four key decisions that are to be made during the design of a dimensional model include:

1. Select the business process - the operational activities done by the business, these activities create the facts

2. Declare the grain - what a single row represents.  The _atomic grain_ is the lowest data captured by the business, which is the ideal and can be aggregared (rolled-up) to other levels.  Different grains must not be mixed in the same fact table


3. Identify the dimensions - the descriptive attributes about the facts, to be used for analysis.  Provide the “who, what, where, when, why, and how” (6W) context 

4. Identify the facts - the measurements (how many) from the business process, it should relate to a physical observable event, rather than reporting needs

Typicall the output of this process is a star schema, with a fact table at the centre supported by the associated dimension tables, with primary/forenigh key relationships.  This is often then structured into a online analytical processing (OLAP) cube, which contains the facts and dimensions appropriate to the analysis, but allows for more detailed analytical capabilities than SQL.  Sometimes aggregated fact tables are built to speed up query performance, as are aggregated OLAP cubes which are typically designed for users.

A key advantage of the dimensional model approach is that new dimensions can be added to an existing fact table by adding a new foreign key column. 

## Graphical Representations

<div class="figure">
<img src="images/StarAndOLAP.png" alt="Star schema versus OLAP cube [@Kimball2013, pg 9]"  />
<p class="caption">(\#fig:StarOLAP)Star schema versus OLAP cube [@Kimball2013, pg 9]</p>
</div>

<div class="figure">
<img src="images/FactWithDims.png" alt="Star schema example [@Kimball2013, pg 16]"  />
<p class="caption">(\#fig:starexample)Star schema example [@Kimball2013, pg 16]</p>
</div>

<div class="figure">
<img src="images/FactWithDimsReport.png" alt="Star schema reporting [@Kimball2013, pg 17]"  />
<p class="caption">(\#fig:starreport)Star schema reporting [@Kimball2013, pg 17]</p>
</div>


## Tips

* Just because something exists in the organisation it does not mean it has to be included.  You need to think about that to include and what not to include.

# References {-}
