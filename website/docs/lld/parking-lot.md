

# Designing Parking Lot

In this artile we will design a parking lot and see its variations as asked in an interview.



##### Functional Requirements 

1. Customers can collect a parking ticket from the entry points and can pay the parking fee at the exit points on their way out.
2. Customers can pay the tickets at the automated exit panel or to the parking attendant.

3. Customers can pay via both cash and credit cards.
4. Customer can pay online in the portal.
5. System should support parking for different types of vehicles.
6. Per hour parking fee model.
   
##### Questions to be asked...

1. Is the parking lot multi floored?  
   YES
2. What type of vehicles are we targeting at ? 
   Car, truck, Electric vehicles(support for nearby charging), Scooters, van, Motorcycle
3. Can we pre book our slot for parking ?
   Lets keep that for phase 2 (Not now)


##### Non Functional Requirements 
1. System should not allow vehicles more than the maximum capacity.
2. Multiple entry and exit points.
3. Multiple parking spots i.e Compact, Large, Handicapped, Motorcycle, etc
4. Each parking floor should have a display board showing any free parking spot for each spot type.

**Note**: Now we have all the things that needs to be done. Lets understand who all are the users of our application.

1. **Admin**: Responsible for adding/ removing/ updating parking floors, parking spots, entrance, and exit panels and parking attendants.
2. **Customer**: Can get a parking ticket and pay for it
3. **Parking Attendant**: Parking attendants can do all the activities on the customer’s behalf, and can take cash for ticket payment.
4. **System**:  To display messages on different info panels, as well as assigning and removing a vehicle from a parking spot.

#### Use Case Diagram

![usecase](../../static/img/parking-lot/usecase.svg)

