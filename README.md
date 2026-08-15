# NestJS-Modules


#### create module
```bash
nest g module [name]
```
<img width="244" height="52" alt="image" src="https://github.com/user-attachments/assets/3897f28d-f884-491a-902d-a7a64162a5bb" />

---



#### create service and controller
```bash
nest g service employee
```
```bash
nest g controller employee
```
<img width="298" height="135" alt="image" src="https://github.com/user-attachments/assets/4d07e2d2-c8c6-4ee5-a1e6-23a1edaa9a97" />

---


#### employee.controller.ts
```bash
# employee.controller.ts
import { Controller, Get } from '@nestjs/common';

@Controller('employee')
export class EmployeeController {
    @Get()
        getEmployee(){
            return 'Employee data fetched successfully!!'
        }
}
```
---


> ### Output
> <img width="358" height="160" alt="image" src="https://github.com/user-attachments/assets/ff1ebd43-684c-4558-bf73-13b14cb7c37f" />



#### employee.service.ts
```bash
import { Injectable } from '@nestjs/common';

@Injectable()
export class EmployeeService {
    private employees = [
        {id: 1101, name: 'Wasim', post: 'Software Engineer'},
        {id: 1102, name: 'Ismail', post: 'DevOps Engineer'},
        {id: 1103, name: 'Pranto', post: 'Network Engineer'},
        {id: 1104, name: 'Omar', post: 'T. Manager'},
    ];
    getAllEmployees(){
        return this.employees;
    }
    getEmployeeById(id: number){
        return this.employees.find((employee) => employee.id === id)
    }
}
```
---


#### employee.controller.ts
```bash
import { Controller, Get, Param } from '@nestjs/common';
import { EmployeeService } from './employee.service';

@Controller('employee')
export class EmployeeController {
    constructor(private readonly employeeService: EmployeeService){}
    @Get()
        getEmployees(){
            return this.employeeService.getAllEmployees();
        }
    @Get(':id')
        getEmployee(@Param('id') id:string){
            return this.employeeService.getEmployeeById(Number(id))
        }
}
```
---
