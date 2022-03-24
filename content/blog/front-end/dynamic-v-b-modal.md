---
title: "v-b-modal 변수와 함께 사용하기"
date: "2022-03-24"
---

## 0. 서론
리스트 페이지에서 각 항목마다 버튼을 배치하고, 클릭시 항목별로 다른 모달이 팝업되도록 하고싶었다.  

## 1. 적용 방법
처음 사용하던 소스는 다음과 같다. `for loop` 안에서 같은 `id`의 모달을 여러번 사용했다. 
```vue
<b-tr v-for="inventory_process in inventory_process_list" :key="inventory_process.id">
...
<b-td class="text-center">
  <b-button variant="danger" v-b-modal.modal-block-vcenter>삭제</b-button>
</b-td>
<!-- Vertically Centered Block Modal -->
<b-modal id="modal-block-vcenter" body-class="p-0" centered hide-footer hide-header>
  <div class="block block-rounded block-themed block-transparent mb-0">
    <div class="block-header bg-primary-dark">
...
</b-tr>
```

검색 후 다음과 같이 변경했다.
```vue
기존 : <b-button variant="danger" v-b-modal.modal-block-vcenter>삭제</b-button>

변경 : <b-button variant="danger" v-b-modal="'modal-block-vcenter-' + inventory_process.id">삭제</b-button>
```

```vue
기존 : <b-modal id="modal-block-vcenter" body-class="p-0" centered hide-footer hide-header>

변경 : <b-modal :id="`modal-block-vcenter-`+inventory_process.id" body-class="p-0" centered hide-footer hide-header>
```

[출처](https://stackoverflow.com/questions/47281857/bind-modal-to-corresponding-button-after-a-v-for)

## 99. 끝나고 나서
`for loop` 내부에 모달을 만들지 말고 하나의 모달에 버튼을 누를때마다 데이터를 전달하는 방식으로 하는것이 더 좋았을것같다. 