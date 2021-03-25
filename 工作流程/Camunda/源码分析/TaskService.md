```java
public interface TaskService {
    
    Task newTask();
    
    
    
    Task newTask(String taskId);
    
    void saveTask(Task task);
    
    
    void deleteTask(String taskId);
    
    
      /**
       * Claim responsibility for a task: 任务的责任链
       * the given user is made {@link Task#getAssignee() assignee} for the task.
       * The difference with {@link #setAssignee(String, String)} is that here
       * a check is done if the task already has a user assigned to it.
       * No check is done whether the user is known by the identity component.
       *
       * @param taskId task to claim, cannot be null.
       * @param userId user that claims the task. When userId is null the task is unclaimed,
       * assigned to no one.
       *
       * @throws ProcessEngineException
       *          when the task doesn't exist or when the task is already claimed by another user.
       * @throws AuthorizationException
       *          If the user has no {@link Permissions#UPDATE} permission on {@link Resources#TASK}
       *          or no {@link Permissions#UPDATE_TASK} permission on {@link Resources#PROCESS_DEFINITION}
       *          (if the task is part of a running process instance).
       */
    void claim(String taskId, String userId);
    
    
    
    
    void resolveTask(String taskId);
}
```

