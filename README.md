# monte-carlo-hospital-sim
Discrete-even simulation (DES) of the patient flow through hospital operating rooms to 
generate a random dataset. __Note that the generated dataset is entirely fabricated__. 
The generated dataset was originally visualized using a particular process visualization
package (such as ProM or Celonis). Due to the proprietary nature this process mining software, 
only the discrete-event simulation is contained within this repository.

### Modelling Patient Arrival Times

* The interval between patient arrivals is estimated based on a user input for the number of patients processed per year. 
    - _Running Example: UIHC's Main OR for FY2019 = 19490. This indicates a rate of 1 patient every 0.45 hours._
* Patient types are broken down into SCHEDULED, FLOOR, or ED TRAUMA by frequency of occurence.
    - _Let's say UIHC is 50% `SCHEDULED`, 30% `FLOOR`, and 20% `ED TRAUMA`._
* Non-emergency (`SCHEDULED` and `FLOOR`) patients are assumed to __arrive__ during business hours (9AM-5PM). There are 2,088 business hours in 2019. This is used to calculate the expected hour intervals between arrivals (`0.13` hours between arrivals).
* Emergency (ED TRAUMA) patients arrive on a 24/7 basis (`2.25` hours per emergency patient)
* The actual arrival intervals are then randomly sampled from the exponential distribution where the scale parameter is set to the expected intervals calculated above.

### Modelling Pre-processing Stages

* Unlike the ORs, there's no capacity limit for `PAT`, `REGISTERED`, `PRE-OP`, and `QUEUE` nodes.
* Due to randomization, patients will travel through these nodes at different rates, so overtaking is possible.
     
### Modelling OR and ER Queues and Operations

* Once they've arrived at the queue, patients are treated __strictly__ in the order of their arrival at the hospital.
    - _E.g. There are 12 ORs. Patient i is the ith patient to arrive. Regardless of how long they take to move through the pre-processing stages, patients 1-12 will __not have to queue__ for ORs 1-12. Let's assume patient 13 moves quickly through pre-processing, arriving at the OR `QUEUE` __before__ patients 1-12. She __still has to queue__, allowing patients 1-12 overtake her into the ORs. Patient 13 takes the next OR that opens up._
* There is at most 1 patient per OR / ER at any one time.
* If there's a queue, the end time of an operation is the start time of the next patient's operation. No lag time accounting for hospital turnover is included.

### Modelling Post-processing Stages

* Unlike the ORs, there's no capacity limit for `POST-OP` nodes.
* The patient's mode of discharge (`DISCHARGED`, `LONG-TERM CARE`, or `FLOOR`), is selected from a matrix of probabilities depending on whether they were `SCHEDULED`, `FLOOR`, or `ED TRAUMA`.