# EMPLOYEE ONBOARDING APP USING REACT,SPRINGBOARD AND SQL
## AIM:
To develop a employee onboarding app using react,spring boot and sql.
## ALGORITHM:
1. Create a springboot project.
2. Add neccessary java files to the project.
3. Connect it with a database.
4. Now create the front end using react.
5. Create required components in react project.
6. Run the project.

## PROGRAM:
### SPRINGBOOT CODE:
### Employee.java
```
package com.saveetha.employee.emp;

import jakarta.persistence.*;

import java.time.LocalDate;
import java.time.Period;

@Entity
@Table
public class Employee {
    @Id
    @SequenceGenerator(
            name="employeeSequence",
            sequenceName = "employeeSequence",
            allocationSize = 1
    )
    @GeneratedValue(
            strategy = GenerationType.SEQUENCE,
            generator = "employeeSequence"
    )
    private Long employeeID;
    private String employeeName;
    @Transient
    private Integer employeeAge;
    private LocalDate dateOfBirth;
    private String employeeEmail;

    public Employee() {
    }

    public Employee(String employeeName, LocalDate dateOfBirth, String employeeEmail) {
        this.employeeName = employeeName;
        this.dateOfBirth = dateOfBirth;
        this.employeeEmail = employeeEmail;
    }





    public Long getEmployeeID() {
        return employeeID;
    }

    public void setEmployeeID(Long employeeID) {
        this.employeeID = employeeID;
    }

    public String getEmployeeName() {
        return employeeName;
    }

    public void setEmployeeName(String employeeName) {
        this.employeeName = employeeName;
    }

    public Integer getEmployeeAge() {
        return Period.between(this.dateOfBirth, LocalDate.now()).getYears();
    }

    public void setEmployeeAge(Integer employeeAge) {
        this.employeeAge = employeeAge;
    }

    public LocalDate getDateOfBirth() {
        return dateOfBirth;
    }

    public void setDateOfBirth(LocalDate dateOfBirth) {
        this.dateOfBirth = dateOfBirth;
    }

    public String getEmployeeEmail() {
        return employeeEmail;
    }

    public void setEmployeeEmail(String employeeEmail) {
        this.employeeEmail = employeeEmail;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "employeeID=" + employeeID +
                ", employeeName='" + employeeName + '\'' +
                ", employeeAge=" + employeeAge +
                ", dateOfJoining=" + dateOfBirth +
                ", employeeEmail='" + employeeEmail + '\'' +
                '}';
    }
}

```
### EmployeeController.java
```
package com.saveetha.employee.emp;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;


import java.util.List;

@RestController
@RequestMapping(path="/api/v1/employee")
public class EmployeeController {
    private final EmployeeService employeeService;
    @Autowired
    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }



    @GetMapping("/")
    public List<Employee> getEmployeeDetails(){
        return employeeService.displayEmployeeDetails();
    }

    @PostMapping("/")
    public void postEmployee(@RequestBody Employee employee){
        employeeService.registerNewEmployee(employee);
    }

    @DeleteMapping(path={"/{id}"})
    public void deleteEmployee(@PathVariable("id") Long employeeId)
    {
        employeeService.removeEmployee(employeeId);
    }
}

```
### EmployeeService.java
```
package com.saveetha.employee.emp;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;


import java.time.LocalDate;
import java.time.Month;
import java.util.List;
import java.util.Optional;

@Component
public class EmployeeService {
    private final EmployeeRepository employeeRepository;
    @Autowired
    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public List<Employee> displayEmployeeDetails() {
        return employeeRepository.findAll();
    }

    public void registerNewEmployee(Employee employee) {
        Optional<Employee> employeeOptional=employeeRepository.findEmployeeByEmail(employee.getEmployeeEmail());
        if(employeeOptional.isPresent()){
            throw new IllegalStateException("Email ID already used");
        }
        employeeRepository.save(employee);
    }

    public void removeEmployee(Long employeeId) {
        boolean employeeExists= employeeRepository.existsById(employeeId);
        if(!employeeExists){
            throw new IllegalStateException("Employee with ID"+employeeId+"doesn't exist");
        }
        employeeRepository.deleteById(employeeId);
    }
}

```
### REACT PROGRAM:
### App.js
```
import React from "react"
import './App.css';
import { BrowserRouter as Router, Route, Routes, Link } from "react-router-dom";
import EmployeeRegistrationComponent from './components/EmployeeRegistrationComponent/EmployeeRegistrationComponent';
import HeaderComponent from "./components/HeaderComponent/HeaderComponent";
import EmployeeDirectoryComponent from "./components/EmployeeDirectoryComponent/EmployeeDirectoryComponent";
import EmployeeDeletionComponent from "./components/EmployeeDeletionComponent/EmployeeDeletionComponent";

function App() {
  return (
    <Router>
            <div className="container">
              <HeaderComponent/>
              
            <nav className="nav-menu">
                <Link to="/" >Home</Link>
                <Link to="/admin/add" >Add Employee</Link>
                <Link to="/admin/delete" >Delete Employee</Link>
            </nav>
           <Routes>
                 <Route exact path='/' element={<EmployeeDirectoryComponent/>}></Route>
                 <Route path='/admin/add' element={<EmployeeRegistrationComponent/>}></Route>
                 <Route path='/admin/delete' element={<EmployeeDeletionComponent/>}></Route>
          </Routes>
          </div>
       </Router>
  );
}

export default App;
```
### EmployeeDirectoryComponent.js
```
import React, { useEffect, useState } from 'react';
import './EmployeeDirectoryComponent.css'; 

function EmployeeDirectoryComponent() {
  const [employees, setEmployees] = useState([]);

  useEffect(() => {
    fetchEmployees();
  }, []);

  const fetchEmployees = async () => {
    try {
      const response = await fetch('http://localhost:8080/api/v1/employee/');
      const data = await response.json();
      setEmployees(data);
    } catch (error) {
      console.error('Error:', error);
    }
  };

  return (
    <div className="employee-list">
      {employees.map((employee) => (
        <div className="employee-card" key={employee.employeeID}>
          <p>Employee ID: {employee.employeeID}</p>
          <p>Name: {employee.employeeName}</p>
          <p>Age: {employee.employeeAge}</p>
          <p>Date of Birth: {employee.dateOfBirth}</p>
          <p>Email: {employee.employeeEmail}</p>
        </div>
      ))}
    </div>
  );
};

export default EmployeeDirectoryComponent;
```
### EmployeeRegistration.js
```
import React, { useState } from 'react';
import './EmployeeRegistrationComponent.css'

function EmployeeRegistrationComponent() {
  const [employee, setEmployee] = useState({
    employeeName: '',
    dateOfBirth: '',
    employeeEmail: ''
  });

  const handleChange = (event) => {
    const { name, value } = event.target;
    setEmployee({ ...employee, [name]: value });
  };

  const handleSubmit = async(event) => {
    event.preventDefault();
    await fetch('http://localhost:8080/api/v1/employee/',{
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(employee)
    })
        .then((response) => {
            if (response.status === 500)
            {
                alert(`Error!`)
            }
            else{
                alert(`Data of ${employee.employeeName} is added successfully`)
                window.location.href = '/'
            }
        })
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Employee Name:
        <input
          type="text"
          name="employeeName"
          value={employee.employeeName}
          onChange={handleChange}
          required
        />
      </label>
      <label>
        Date of Birth:
        <input
          type="date"
          name="dateOfBirth"
          value={employee.dateOfBirth}
          onChange={handleChange}
          required
        />
      </label>
      <label>
        Employee Email:
        <input
          type="email"
          name="employeeEmail"
          value={employee.employeeEmail}
          onChange={handleChange}
          required
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
};

export default EmployeeRegistrationComponent;

```
### EmployeeDeletionComponent.js
```
import React, { useState } from 'react';
import './EmployeeDeletionComponent.css'

function EmployeeDeletionComponent() {
  const [employeeID, setEmployeeID] = useState('');

  const handleChange = (event) => {
    setEmployeeID(event.target.value);
  };

  const handleSubmit = async(event) => {
    event.preventDefault();
    await fetch(`http://localhost:8080/api/v1/employee/${employeeID}`,{
      method: 'DELETE'
    })
    .then((response) => {
            if (response.status === 500)
            {
                alert(`Error!`)
            }
            else{
                alert(`Data deleted successfully`)
                window.location.href = '/'
            }
        })
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Employee ID:
        <input
          type="number"
          name="employeeID"
          value={employeeID}
          onChange={handleChange}
          required
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
};

export default EmployeeDeletionComponent;

```
## OUTPUT:
![image](https://github.com/Rithigasri/EmployeeManagement/assets/93427256/f007b2b3-17b7-4c7c-8710-82e2b9ee8e61)
![image](https://github.com/Rithigasri/EmployeeManagement/assets/93427256/8b213066-90aa-4a2e-b1b6-28681852a768)
![image](https://github.com/Rithigasri/EmployeeManagement/assets/93427256/d0ec6f75-882e-4ecc-b29b-af6fa208bb90)
## RESULT:
Thus, a employee onboarding app is developed using springboot,sql and react.
