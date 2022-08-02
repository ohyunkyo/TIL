---
title: "모달에 데이터 전달하기"
date: "2022-03-31"
---

## 0. 서론
frontend 와 backend 를 분리하기 이전, django 템플릿으로 리스트 페이지를 만들고 각각의 객체에 수정과 삭제 버튼을 추가한 모델이 있었다.  
수정 버튼은 수정페이지로 이동하고 삭제 버튼은 안내메시지를 포함한 모달을 띄운다.  
이때 각 안내메시지가 해당 객체에 대한 내용을 포함하도록 하고 싶었기 때문에 객체마다 개별적인 모달을 가지고 있었고
매우 비효율적이라는 생각이 들어 나중에 분리할 방법을 찾아 수정할 예정이였다.

```html
# inventory_process_list.html

{% if inventory_process_list %}
    {% for inventory_process in inventory_process_list %}
        <tr>
            <td>{{ inventory_process.name }}</td>
            <td>{% effect_check inventory_process.effect %}</td>
            <td class="center"><a href="{% url 'inventory:inventory_process_update' inventory_process.id %}" class="btn btn-update">수정</a></td>
            <td class="center"><a href="#modal-inventory-process-delete-{{ inventory_process.id }}" class="btn btn-delete modal-trigger">삭제</a></td>
        </tr>
        <!-- Modal Structure -->
        <div id="modal-inventory-process-delete-{{ inventory_process.id }}" class="modal">
            <form id="delete-form-{{ inventory_process.id }}" method="post" action="{% url 'inventory:inventory_process_delete' inventory_process.id %}">
                {% csrf_token %}
                <div class="modal-content">
                    <h4 class="red-text accent-4">재고처리 방법을 삭제합니다</h4>
                    <p>"예" 를 누르면 다음 항목이 삭제됩니다.<br>
                        재고처리명 : <span class="black-text font-size-22 bold">{{ inventory_process.name }}</span><br>
                        재고수량처리방법 : <span class="black-text font-size-22 bold">{% effect_check inventory_process.effect %}</span><br>
                        정말로 삭제하시겠습니까?</p>
                </div>
                <div class="modal-footer">
                    <a href="#" class="modal-close waves-effect waves-green btn-flat red-text accent-3">아니오</a>
                    <button type="submit" class="modal-close waves-effect waves-green btn-flat">예</button>
                </div>
            </form>
        </div>
    {% endfor %}
{% endif %}
```

이후 vue.js / drf 로 분리하면서 모달 하나에 삭제할 객체의 정보를 전달하여 출력하는 식으로 변경하고자 방법을 찾아보게 되었다.

## 1. 데이터를 전달하는 방법
데이터를 전달하는 방법은 아주 간단하다. 반복문 내부에 모달을 배치하는것보다 훨씬 쉽고 편리하다.  
모달 내부에 변수를 출력하도록 하고 모달을 띄우는 버튼을 클릭할 때마다 해당 변수에 값을 넣어주면 되는것이다.
```html
# 예제 - html 

<div id="app">
  <ul>
    <li v-for="item in items">
      {{ item.first_name }}
      <b-button size="sm" v-b-modal="'myModal'" user="'item'" click="sendInfo(item)">
        Saluta {{item.first_name}}
      </b-button>
    </li>
  </ul>

  <b-modal id="myModal">
    Hello {{selectedUser.first_name}} {{selectedUser.last_name}} !
  </b-modal>
</div>
```
```javascript
# 예제 - javascript

new Vue({
  el: '#app',
  data: {
    items :
    [
        { first_name: 'Dickerson', last_name: 'Macdonald' },
        { first_name: 'Larsen', last_name: 'Shaw' },
        { first_name: 'Geneva', last_name: 'Wilson' },
        { first_name: 'Jami', last_name: 'Carney' }
    ],
    selectedUser: '',
  }, 
  methods: {
    sendInfo(item) {
        this.selectedUser = item;
    }
  }
})
```
[출처](https://stackoverflow.com/questions/51271337/bootstrap-vue-how-to-pass-data-to-modal)

## 2. 모달 분리하기
for loop 내부에 있던 모달을 반복문 밖으로 빼내고 안내메시지를 변수화 했다.

```vue
<!-- Vertically Centered Block Modal -->
<b-modal id="delete-inventory-process-modal" body-class="p-0" centered hide-footer hide-header>
  <div class="block block-rounded block-themed block-transparent mb-0">
    <div class="block-header bg-primary-dark">
      <h3 class="block-title">{{ delete_object.modal_title }}</h3>
      <div class="block-options">
        <button type="button" class="btn-block-option" @click="$bvModal.hide('delete-inventory-process-modal')">
          <i class="fa fa-fw fa-times"></i>
        </button>
      </div>
    </div>
    <div class="block-content font-size-sm">
      <p>
        "예" 를 누르면 <span class="lead font-w700 bg-warning text-gray-lighter">&nbsp;{{ delete_object.object.name }}&nbsp;</span> {{ delete_object.modal_message }}
      </p>
      <p>
        정말로 삭제 하시겠습니까?
      </p>
    </div>
    <div class="block-content block-content-full text-right border-top">
      <b-button variant="alt-primary" class="mr-1" @click="$bvModal.hide('delete-inventory-process-modal')">아니오</b-button>
      <b-button variant="alt-danger" @click="delete_inventory_process(delete_object.model, delete_object.object.id)">예</b-button>
    </div>
  </div>
</b-modal>
<!-- END Vertically Centered Block Modal -->
```

## 2. 변수에 데이터 전달하기
클릭시 데이터를 전달하기 위한 메서드가 실행되도록 버튼을 수정했다 
```vue
...
<b-button variant="danger" v-b-modal.delete-inventory-process-modal @click="setModalData('inventory_process', inventory_process)">삭제</b-button>
...
```
안내메시지 설정을 위한 model 이름, 삭제 요청을 위한 id(pk), 삭제 확인을 위한 name 을 modal 에 전달한다. 
```javascript
...
methods:{
    setModalData(model, object){
        if (model === 'inventory_process') {
            this.delete_object.modal_title = '재고 처리 방법을 삭제합니다.'
            this.delete_object.modal_message = '재고 처리 방법이 삭제됩니다.'
        }

        this.delete_object.model = model
        this.delete_object.object = object
    }
}
...
```