Introduction

    The aim of this lab is to focus on synchronization of threads over a bus. This bus is a half-duplex communication one, that means we can only send data in one way. The other constraints to take under
    consideration are the following: the bus has limited space for threads and there are 2 types of priority, normal and priority. To answer this problem we have modified the functions init_bus(), get_slot()
    and release_slot().
    The first function is used to initialise the variables we need.
    The get_slot() function permits a thread to go in the bus. This function is managing synchronization problem and give the priority to the tasks which have it.
    Moreover, this function manages also the way in which the threads are going.
    Lastly, release_slot() is necessary to release a slot once data has been transferred. This function is also used to modify the direction of data flow when the buffer is empty.
  
Deciding on how to work on the assignment

    After the introduction of the lab in class our group decided it was best to try to implement the lab separately because of the warnings from the TA. The TA’s warnings were that if different parts were implemented by
    different group members the final program might not run correctly because of compatibility issues. Each member has worked on the lab and every 2-3 days our group has had meetings to discuss our implementation, ideas
    and problems we encountered. After each meeting we decided on which code was best to use for the rest of the implementation. This way each member has been working on the whole lab, not just parts of it. The final 
    implementation of the lab builds a lot from the producer-consumer problem from the video on canvas. After the code was finalized, we had a meeting writing the report together.

Data structures and fields

    We have introduced some data structures, local variables and few conditional variables for the functionality of the code. Some of them are listed as below:
    Struct lock bus_lock
    This structure is used to protect the critical section of the code specially for the synchronization purpose. This lock is used to ensure that only one task can try to acquire a slot on the bus at a time.
    Struct condition not_empty
    This condition variable is used for signalling and waiting by tasks that have acquired the slots on the bus. It is used for managing the release of slots.
    Struct condition has_space_send
    The tasks use this condition variable if they want to send data. This condition variable helps to manage slot availability for sending tasks based on their direction.
    Struct condition has_space_receive
    This is used for tasks that want to receive data. This condition helps to manage the slot availability for receiving tasks based on their direction.
    Int send_prios
    This variable is incremented when a priority task of sending direction is trying to access the bus and decremented for when a priority task enters the bus.
    Int receive_prios
    It is incremented for when a priority task of receiving direction is trying to access the bus and decremented for when a priority task enters the bus.
    Int active_tasks 
    This integer variable is used to maintain the count of the total number of tasks currently using the bus. It is incremented when a task successfully acquires a slot on the bus and
    decremented when a task releases the slot.


The algorithm

    The goal of the algorithm is to synchronize tasks over a half-duplex bus, information can only be either sent or received over the bus at any given time. The implementation relies heavily on conditional variables,
    locks and counters. First the bus is initialized with a call to init_bus(). After a task is dispatched it tries to acquire a slot on the bus. Before it gets the slot it needs to pass some checks to see if it is
    allowed a slot. If the bus is at its maximum capacity, 3 tasks, or if the current direction of the bus is different from that of the task while some other task is on the bus then it has to conditionally wait for
    there to be space on the bus. Also, if the task has normal priority, it has to wait until there are no more priority tasks waiting for a slot on the bus. If it is a priority task then the respective counter gets
    incremented by 1, showing that some priority task is waiting to acquire a slot. When entering the bus a signal is sent to say that there are tasks on the bus that can be removed. The direction of the bus is also
    updated to be that of the entering task. After this the task does its transfer of data, which is sleeping. If a priority task enters the bus the respective counter is decremented by 1, showing that one less priority 
    task is waiting to acquire a slot. The last thing to happen is to release its slot. If there are no tasks to be removed then the consumer has to wait until there is something on the bus to remove. After the wait, 
    depending on the direction of the task, it either signals another task from the same direction that it should try to acquire a slot or, if there are no more tasks from that side, it broadcasts to all the tasks of
    the other direction that the bus is empty and to fill it up.
  
    The layout of the algorithm is like the following pseudocode:
    
        init_bus(){
          lock_init(bus_lock);
          cond_init(not_empty);
          …
          receive_prios = 0;
          bus_direction = NUM_OF_DIRECTIONS;
        }
        
        get_slot(){
          acquire_lock(bus_lock);
          while(not allowed a slot){
            cond_wait(space from my direction);
          }	
          	direction = my direction;
          	active_tasks++;
          	signal(not empty);
          	release_lock(bus_lock);
        }
        
        release_slot(){
        	acquire_lock(bus_lock);
        	while(no tasks on the bus){
        		cond_wait(not empty);
        	}
        	active_tasks--;
        	switch(direction of task){
        
        		case SEND:
        			if(active_tasks == 0){
        				cond_broadcast(other direction);
        			}
        			cond_signal(same direction);
            case RECEIVE:
        		  if(active_tasks == 0){
        				cond_broadcast(other direction);
        			}
        		  cond_signal(same direction);
          }  
        	release_lock(bus_lock);
        }


Is the implementation unfair? How to manage this issue?

    This implementation may not be fair in a way. Say, in one direction, if the task is only using the bus for sending tasks then the other direction tasks will have to keep waiting until the sending side priority
    tasks have completed their tasks on the bus. If say, the other direction is receiving tasks, then this other direction receiving tasks will have to keep on waiting for their turn to use the bus until sending
    priority tasks has done their job completely on the bus. Only then the bus will be free and the receiving priority tasks will get their chance on the bus. This way of implementation consumes time and causes
    starvation on the bus.
    Therefore, in order to face this kind of problem, we can maybe use a method such as traffic lights. Just like in traffic lights signal, there is allocated time for vehicles to access their direction, we can also 
    refer a system in which each task is timed for a certain period of time so that each task will get their chance on the bus to complete their task. As for the priority task, they will be given the access to the bus 
    before the other normal tasks just as giving priority to an ambulance on the road even if there are other vehicles lined up. The priority tasks will always have a “green light” on the bus if no other priority tasks 
    from the other direction wants to access the bus. This way, after the priority tasks are done then other normal task can also get their access on the bus just like the vehicles on the road, that will get their
    chance to go to their direction after the ambulance has been released. So, to sum up we can say that, we should use multiplexing based on time for the efficient functioning of the tasks on the bus. 

Race condition in the test

    There is a race condition in the test of the batch-scheduler. The test compares the output from the code with an expected output. The problem is in the run_task() function. In this function the calls are:
    get_slot()
    transfer_data()
    release_slot()
    After the call to get_slot() there is also a call to msg() which prints what task got a slot, since this call is not in a critical section a task that completed get_slot() may be preempted before the call to msg(),
    then another task may call msg() before the first task. This could make the output different than what is expected, the first task to get a slot may not be the first task to output the msg, creating a race condition.
    To verify that the tasks enter the bus in the right order there is a bookkeeping variable, enter_id, that can be printed out within the critical section of get_slot(). This shows the order of the slots being 
    acquired by the tasks.
