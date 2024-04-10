# -AppSecCloudCamp
Часть 2: Security code review: Python
Пример №2.1

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
    
  В строках /name = request.values.get('name')/ и /age = request.values.get('age', 'unknown')/
Присутствует уязвимость в обработке данных запроса. Метод request.values.get() получает данные из строки запроса,
если введеные данные не будут достачно защищены, это может привести к межсайтовой подделки запроса.

  Следующие две потенциальных уязвимости находятся в строке /app.run(debug=True)/.
Первая, лишняя доступность. При стандартном запуске app.run(), сервис доступен для всех IP-адресов, что потенциально создает возможность доступа для злоумышленников.
Вторая, включенный режим отладки. Параметр debug=True включает режим отладки, что может быть опасно в рабочей среде, поскольку раскрывает чувствительные данные.


================================================================================================================================================================

Пример №2.2

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

    
В данном примере в строке /hostname = request.values.get('hostname')/ отсутствует обработка неправильных входных данных. В ситуации когда параметр hostname отсутствует или имеет некорректное значение это может приводить к неправильному выполнеию и уязвимости безоасности.
В строке /output = subprocess.check_output(cmd, shell=True, text=True)/ значение параметра shell=True представляет риск, поскольку это может привести к возможности инъекций команд, особенно если hostname формируется пользователем. 


    
