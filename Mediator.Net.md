Mediator.Net是一个按照Mediator Pattern的开源框架, 通常的程序中会有大量的类, 每个类有不同的封装和参数, 随着一个项目的规模增大, 类之间的互相交互变得极其复杂, 变得难以维护. 所以Mediator Pattern就在于对象之间的通信封装了一个对象, 这个对象就是中介.

![image](https://github.com/satoi-fish/CodeNote/assets/81409285/9eb25fcc-7809-4a34-974c-31fbba85c2b0)

![image](https://github.com/satoi-fish/CodeNote/assets/81409285/3d9aa1f6-ef13-43ea-a2f2-6f377e14b556)

## **运用mediator**
首先在controller层使用SendAsync/RequestAsync,随后选择到对应的管道,对应的handler, 执行任务.
**controller发一个SendAsync**
```
[HttpPost]  
[Route("create")]  
public async Task<IActionResult> CreateAsync([FromBody] CreatePeopleCommand command)  
{  
    var response = await _mediator.SendAsync<CreatePeopleCommand, CreatePeopleResponse>(command)  
        .ConfigureAwait(false);  
  
    return Ok(response);  
}
```
**根据对应的管道找到对应的handler**,里面调用了AddPersonAsync
```
public async Task<CratePeopleResponse> Handle(IReceiveContext<CreatePeopleCommand> context,  
    CancellationToken cancellationToken)  
{  
    var @event = await _personService.AddPersonAsync(context.Message, cancellationToken).ConfigureAwait(false);  
  
    await context.PublishAsync(@event, cancellationToken).ConfigureAwait(false);  
  
    return new CratePeopleResponse  
    {  
        Result = @event.Result  
    };  
}
```
**AddPersonAsync**, 再在里面执行操作
```
public async Task<PeopleCreatedEvent> AddPersonAsync(CreatePeopleCommand command,  
    CancellationToken cancellationToken)  
{  
    return new PeopleCreatedEvent  
    {  
        result = await _personDataProvider.CreatAsync(command.person, cancellationToken).ConfigureAwait(false) > 0  
            ? "数据写入成功"  
            : "数据写入失败"  
    };  
}  
  
public async Task<int> CreatAsync(Person person, CancellationToken cancellationToken)  
{  
    await _dbContext.People.AddAsync(person, cancellationToken).ConfigureAwait(false);  
  
    return await _dbContext.SaveChangesAsync(cancellationToken).ConfigureAwait(false);  
}
```
参考查阅了:[lizzie学习笔记](https://github.com/DOGGIE4/desktop-tutorial/blob/a9eb7f41ea01b136ab329d5c543c8dc4761b269c/Mediator%20Learn.md)
