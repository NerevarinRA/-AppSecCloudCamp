# -AppSecCloudCamp
Часть 2: Security code review: Python
Пример №2.1

```python
from flask import Flask, request
from jinja2 import Template

app = Flask(name)

@app.route("/page")
def page():
    name = request.values.get('name')
    age = request.values.get('age', 'unknown')
    output = Template('Hello ' + name + '! Your age is ' + age + '.').render()
return output

if name == "main":
    app.run(debug=True)
```
    
  В строках ```name = request.values.get('name')``` и ```age = request.values.get('age', 'unknown')```
присутствует уязвимость в обработке данных запроса. Метод request.values.get() получает данные из строки запроса,
если введеные данные не будут достачно защищены, это может привести к межсайтовой подделки запроса(CSRF).  
Для утранения данной уязвимости в ```Flask``` есть возможность использовать расширение ```Flask-WTF``` для генерации и проверки токенов CSRF.

Следующая уязвимость находятся в строке ```app.run(debug=True)```.  
Параметр ```debug=True``` включает режим отладки, что может быть опасно в рабочей среде, поскольку раскрывает чувствительные данные.  
Перед развертыванием в рабочей среде нужно отключить режим отладки среде изменив параметр ```debug=True``` на ```debug=False```.  


===============================================================================================================  

Пример №2.2

```python
from flask import Flask, request
import subprocess

app = Flask(name)

@app.route("/dns")
def dns_lookup():
    hostname = request.values.get('hostname')
    cmd = 'nslookup ' + hostname
    output = subprocess.check_output(cmd, shell=True, text=True)
return output
if name == "main":
    app.run(debug=True)
```
    
В данном примере в строке ```hostname = request.values.get('hostname')``` отсутствует обработка неправильных входных данных. В ситуации когда параметр hostname отсутствует или имеет некорректное значение это может приводить к неправильному выполнеию и уязвимости безоасности.  
Достаточно будет добавить проверку наличия и корректности параметра ```hostname``` в функции ```dns_lookup```.   

В строке ```output = subprocess.check_output(cmd, shell=True, text=True)``` значение параметра ```shell=True``` указывает, что команда cmd должна выполняться через командную оболочку операционной системы. Это является риском, поскольку может привести к возможности инъекций команд, особенно если ```hostname``` формируется пользователем.  

Уязвимость строки ```app.run(debug=True)``` была рассмотренна в предыдущем примере.  

======================================================================================================================

Часть 1. Security code review: GO  
```GO
package main

import (
    "database/sql"
    "fmt"
    "log"
    "net/http"
    "github.com/go-sql-driver/mysql"
)

var db *sql.DB
var err error

func initDB() {
    db, err = sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        log.Fatal(err)
    }

err = db.Ping()
if err != nil {
    log.Fatal(err)
    }
}

func searchHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        http.Error(w, "Method is not supported.", http.StatusNotFound)
        return
    }

searchQuery := r.URL.Query().Get("query")
if searchQuery == "" {
    http.Error(w, "Query parameter is missing", http.StatusBadRequest)
    return
}

query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)
rows, err := db.Query(query)
if err != nil {
    http.Error(w, "Query failed", http.StatusInternalServerError)
    log.Println(err)
    return
}
defer rows.Close()

var products []string
for rows.Next() {
    var name string
    err := rows.Scan(&name)
    if err != nil {
        log.Fatal(err)
    }
    products = append(products, name)
}

fmt.Fprintf(w, "Found products: %v\n", products)
}

func main() {
    initDB()
    defer db.Close()

http.HandleFunc("/search", searchHandler)
fmt.Println("Server is running")
log.Fatal(http.ListenAndServe(":8080", nil))
}
```
Строка ```query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)``` представляет  
представляет собой слияние строки запроса с пользовательским вводом ```searchQuery```. Это является уязвимостью к SQL-инъекциям, так как ```searchQuery``` может содержать зловредные SQL-команды.  
Для предотвращения SQL-инъекций можно использовать запросы с параметрами, что позволит передавать пользовательские данные как параметры запроса вместо ввода напрямую.  

В строке ```db, err = sql.Open("mysql", "user:password@/dbname")``` пароль указан в явном виде.  
Это небезопасно, так как пароль может быть увиден в коде.  
Для хранения конфиденциальных данных, таких как пароли можно использовать хэширование. 

Так же в коде не реализованы методы шифрования, что делает передаваемые данные уязвимыми к перехвату.  
Эту проблему можно решить переходом с протокола ```http``` на ```https``` для шифрования трафика.  

В коде отсутствует проверка доступа к ресурсу ```/search```, что может привести к утечке конфиденциальной информации из базы данных.  
Стоит реализовать механизм аутентификации пользователей.



    
