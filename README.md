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



####
```bash

```
---
