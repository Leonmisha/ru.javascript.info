
# FormData

В этой главе речь пойдёт об отправке HTML-форм: с файлами и без, с дополнительными полями и так далее. Объекты [FormData](https://xhr.spec.whatwg.org/#interface-formdata) могут помочь нам в решении этих задач.

Конструктор:
```js
let formData = new FormData([form]);
```

<<<<<<< HEAD
Если передать в конструктор элемент HTML-формы `form`, то создаваемый объект автоматически прочитает из неё поля. Как вы, наверно, уже догадались, `FormData` - объект для хранения и отправки данных формы.

Его особенность заключается в том, что методы для работы с сетью, например `fetch`, позволяют указать объект `FormData` в свойстве `body` запроса. Он будет соответствующим образом закодирован и отправлен с заголовком `Content-Type: form/multipart`.

Таким образом, со стороны сервера это выглядит как обычная отправка формы.
=======
If HTML `form` element is provided, it automatically captures its fields. As you may have already guessed, `FormData` is  an object to store and send form data.

The special thing about `FormData` is that network methods, such as `fetch`, can accept a `FormData` object as a body. It's encoded and sent out with `Content-Type: form/multipart`. So, from the server point of view, that looks like a usual form submission.
>>>>>>> be342e50e3a3140014b508437afd940cd0439ab7

## Отправка простой формы

Давайте сначала отправим простую форму.

Как вы видите, код очень компактный:

```html run autorun
<form id="formElem">
  <input type="text" name="name" value="John">
  <input type="text" name="surname" value="Smith">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user', {
      method: 'POST',
*!*
      body: new FormData(formElem)
*/!*
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

В этом примере сервер принимает POST-запрос с данными формы и отвечает сообщением "Пользователь сохранён.".

## Методы объекта FormData

С помощью указанных ниже методов мы можем изменять поля в объекте `FormData`:

- `formData.append(name, value)` - добавляет к объекту поле с именем `name` и значением `value`,
- `formData.append(name, blob, fileName)` - добавляет поле, как будто в форме имеется элемент `<input type="file">`, третий аргумент `fileName` устанавливает имя файла (не имя поля формы), как будто это имя из файловой системы пользователя,
- `formData.delete(name)` - удаляет поле с заданным именем `name`,
- `formData.get(name)` - получает значение поля с именем `name`,
- `formData.has(name)` - если существует поле с именем `name`, то возвращает `true`, иначе `false`

Технически форма может иметь много полей с одним и тем же именем `name`, поэтому несколько вызовов `append` добавят несколько полей с одинаковыми именами.

Ещё существует метод `set`, его синтаксис такой же, как у `append`. Разница в том, что `.set` удаляет все уже имеющиеся поля с именем `name` и только затем добавляет новое. То есть этот метод гарантирует, что будет существовать только одно поле с именем `name`:

- `formData.set(name, value)`,
- `formData.set(name, blob, fileName)`.


Поля объекта `formData` можно перебирать, используя цикл `for..of`:

```js run
let formData = new FormData();
formData.append('key1', 'value1');
formData.append('key2', 'value2');

// Список пар ключ/значение
for(let [name, value] of formData) {
  alert(`${name} = ${value}`); // key1=value1, потом key2=value2
}
```

## Отправка формы с файлом

<<<<<<< HEAD
Объекты `FormData` всегда отсылаются с заголовком `Content-Type: form/multipart`, этот способ кодировки позволяет отсылать файлы. Таким образом, поля `<input type="file">` тоже отправляются, как это и происходит в случае обычной формы.
=======
The form is always sent as `Content-Type: form/multipart`, this encoding allows to send files. So, `<input type="file">` fields are sent also, similar to a usual form submission.
>>>>>>> be342e50e3a3140014b508437afd940cd0439ab7

Пример такой формы:

```html run autorun
<form id="formElem">
  <input type="text" name="firstName" value="John">
  Картинка: <input type="file" name="picture" accept="image/*">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user-avatar', {
      method: 'POST',
*!*
      body: new FormData(formElem)
*/!*
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

<<<<<<< HEAD
## Отправка формы с Blob-данными

Ранее в главе <info:fetch> мы видели, что отправить динамически сгенерированные бинарные данные `Blob`, например картинку, достаточно легко. Мы можем явно передать её в параметр `body` запроса `fetch`.

Но на практике бывает удобнее отправлять изображение не отдельно, а в составе формы, добавив дополнительные поля для имени "name" и другие метаданные.
=======
## Sending a form with Blob data

As we've seen in the chapter <info:fetch>, sending a dynamically generated `Blob`, e.g. an image, is easy. We can supply it directly as `fetch` parameter `body`.

In practice though, it's often convenient to send an image not separately, but as a part of the form, with additional fields, such as "name" and other metadata.
>>>>>>> be342e50e3a3140014b508437afd940cd0439ab7

Кроме того, серверы часто настроены на приём именно форм, а не просто бинарных данных.

В примере ниже с помощью объекта `FormData` в составе формы отсылается изображение из `<canvas>` и ещё несколько полей:

```html run autorun height="90"
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Отправить" onclick="submit()">

  <script>
    canvasElem.onmousemove = function(e) {
      let ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      let imageBlob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));

*!*
      let formData = new FormData();
      formData.append("firstName", "John");
      formData.append("image", imageBlob, "image.png");
*/!*    

      let response = await fetch('/article/formdata/post/image-form', {
        method: 'POST',
        body: formData
      });
      let result = await response.json();
      alert(result.message);
    }

  </script>
</body>
```

Пожалуйста, обратите внимание на то, как добавляется изображение `Blob`:

```js
formData.append("image", imageBlob, "image.png");
```

<<<<<<< HEAD
Это как если бы в форме был элемент `<input type="file" name="image">` и пользователь прикрепил бы файл `image.png` (3й аргумент) из своей файловой системы.
=======
That's same as if there were `<input type="file" name="image">` in the form, and the visitor submitted a file named `image.png` (3rd argument) from their filesystem.
>>>>>>> be342e50e3a3140014b508437afd940cd0439ab7

## Итого

<<<<<<< HEAD
Объекты [FormData](https://xhr.spec.whatwg.org/#interface-formdata) используются, чтобы прочитать данные HTML-формы и отправить их с помощью `fetch` или другого метода для работы с сетью.
=======
[FormData](https://xhr.spec.whatwg.org/#interface-formdata) objects are used to capture HTML form and submit it using `fetch` or another network method.
>>>>>>> be342e50e3a3140014b508437afd940cd0439ab7

Мы можем создать такой объект уже с данными, передав в конструктор HTML-форму -- `new FormData(form)`, или же можно создать пустой объект и затем добавить к нему поля с помощью методов:

- `formData.append(name, value)`
- `formData.append(name, blob, fileName)`
- `formData.set(name, value)`
- `formData.set(name, blob, fileName)`

Тут есть две особенности:
1. Метод `set` удаляет предыдущие поля с таким же именем, а `append` -- нет.
2. Чтобы послать файл, нужно использовать методы с тремя аргументами, где в качестве третьего как раз указывается имя файла, которое обычно, при `<input type="file">`, берётся из файловой системы.

Другие методы:

- `formData.delete(name)`
- `formData.get(name)`
- `formData.has(name)`

Это всё!