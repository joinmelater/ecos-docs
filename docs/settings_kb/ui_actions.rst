============
**Действия**
============

ECOS Actions
------------
Описание работы действий в ECOS

.. list-table:: Сущности в системе
      :widths: 10 10 40
      :header-rows: 1

      * - Имя
        - Тип
        - Описание
      * - id
        - String
        - Идентификатор действия. Уникальный среди всех действий в системе
      * - key
        - String
        - Ключ, по которому возможна фильтрация. Должен быть в формате word0.word1.word2 чтобы можно было фильтровать по маске.
      * - name
        - String
        - Имя действия, которое увидит пользователь
      * - type
        - String
        - Тип действия. Тип определяет логику, которая будет выполнена при выполнении действия.
      * - icon
        - String
        - Иконка действия. Пример "icon-delete", "icon-on". Все иконки можно посмотреть в citeck/ecos-ui/src/fonts/citeck/demo.html
      * - config
        - JsonObject
        - Конфигурация действия. Полезно в случаях, когда один тип действия может на основе конфигурации менять свое поведение. Например - для действия с типом Download можно задать шаблон URI для скачивания контента.
      * - predicate
        - Predicate
        - Используется для динамического определения доступности действия для пользователя. Например, действия "Редактировать" и "Удалить" не могут выполнять пользователи без прав на запись и для них эти действия скрываются

Расширение списка действий
~~~~~~~~~~~~~~~~~~~~~~~~~~

Действия - это артефакты ECOS в формате json или yaml с типом ui/action:

Одно действие может быть многократно использовано в разных местах системы (например, в журнале и на карточке документа).

Получение действий по записи
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Для запроса действий отправляется следующий запрос::

 {
    "query": {
        "records": [
            "workspace://SpacesStore/123123-123-123",
            "workspace://SpacesStore/123123-123-124"
        ],
        "actions": [
            "ui/action$delete",
            "ui/action$edit"
        ]
    }
 }

Овтет::

 [
    {
        "record": "workspace://SpacesStore/123123-123-123",
        "actions": [
            {
                "icon": "edit",
                "key": "...",
                "type": "mutate",
                "config": {}
            },
            {
                "icon": "delete",
                "key": "...",
                "type": "delete",
                "config": {}
            }
        ]
    },
    {
        "record": "workspace://SpacesStore/123123-123-124",
        "actions": [
            {
                "icon": "edit",
                "id": "...",
                "type": "mutate",
                "config": {}
            },
            {
                "icon": "delete",
                "id": "...",
                "type": "delete",
                "config": {}
            }
        ]
    }
 ]

Так же доступен вариант раздельного указания действий по записям::

 {
    "query": {
        "records": [
            {
                "record": "workspace://SpacesStore/123123-123-123",
                "actions": [
                    "ui/action$delete",
                    "ui/action$edit"
                ]
            },
            {
                "record": "workspace://SpacesStore/123123-123-555",
                "actions": [
                    "ui/action$edit"
                ]
            }
        ]
    }
 }

Фронтенд
~~~~~~~~

На фронтенде действия описаны в виде javascript сущностей с методами
**execForRecord, execForRecords, execForQuery, getDefaultModel, canBeExecuted** и др.
Например: **src/components/Records/actions/handler/executor/CreateAction.js**
При выполнении действия вызывается метод execute в который передается запись, над которой выполняется действие и конфигурация действия.
Реестр действий описан в **src/components/Records/actions/RecordActionExecutorsRegistry.js**
Регистрация действий в реестре: **src/components/Records/actions/index.js**

Типы действий
-------------

.. list-table::
      :widths: 10 10 40
      :header-rows: 1

      * - Тип
        - Конфигурация
        - Описание
      * - create
        - | ``typeRef: String``
          | ECOS тип для создания. Обязательный параметр;
          | ``createVariantId: String``
          | Идентификатор варианта создания для типа. 
          | Если не указан, то используется первый доступный вариант
          | ``createVariant: Object``
          | Вариант создания для ситуаций, когда ни один 
          | вариант создания из типа не походит и 
          | требуется его полностью определить в действии
          | ``attributes: Object``
          | Предопределенные атрибуты для создания новой сущности. 
          | Для прокидывания атрибутов с текущей записи 
          | (т.е. той, с которой выполняется действие) на форму создания 
          | можно использовать вставки вида ``${attribute_name}``
          | ``options: Object``
          | Опции формы
        - | Действие для создания нового документа. Обычно применяется 
          | когда требуется создать новый документ, в котором некоторые поля 
          | будут предзаполнены из данных текущего открытого документа.

Расширение действий
-------------------

Добавление новых инстансов действий
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Для добавления новых инстансов действий необходимо описать их в json виде и добавить их в alfresco (в микросервисы так же можно добавлять действия) по пути

**{alfresco_module_id}/src/main/resources/alfresco/module/{alfresco_module_id}/ui/action**

Пример описания::

 {
    "id": "confirm-list-html",
    "key": "card-template.confirm-list.html",
    "name": "Скачать лист согласования",
    "type": "download-card-template",
    "config": {
        "templateType": "confirm-list",
        "format": "html"
    }
 }

Для тестирования можно заливать эту конфигурацию в журнале действий вручную.

Добавление новых типов действий
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
На данный момент все типы описаны в базовом проекте ecos-ui (в планах есть поддержка расширения действий без изменений в ecos-ui).

Описываем новое действие::

 export const DownloadAction = {
  execute: ({ record, action }) => {
    const config = action.config || {};

    let url = config.url || getDownloadContentUrl(record.id);
    url = url.replace('${recordRef}', record.id); // eslint-disable-line no-template-curly-in-string

    const name = config.filename || 'file';

    const a = document.createElement('A', { target: '_blank' });

    a.href = url;
    a.download = name;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);

    return false;
  },

  getDefaultModel: () => {
    return {
      name: 'grid.inline-tools.download',
      type: 'download',
      icon: 'icon-download'
    };
  },

  canBeExecuted: ({ record }) => {
    return record.att('.has(n:"cm:content")') !== false;
  }
 };

Зарегистрировать новый тип::

 import Registry from './RecordActionExecutorsRegistry';
 import { DownloadAction } from './DefaultActions';

 Registry.addExecutors({
  download: DownloadAction,
 });

Настройки списка действий
-------------------------
Настройка действий на dashboard
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Настройка действий на dashboard осуществляется в журнале типов кейсов, который располагается в системных журналах:

.. image:: _static/Action_settings.png
       :align: center
       :alt: Настройка действий

Настройка действий в журналах
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Действия в журнале описываются в разделе actions перед headers и содержат ссылки на те же действия, что и в типах. Если действия не описаны, то используется список действий по умолчанию:

ui/action$content-download
ui/action$edit
ui/action$delete
ui/action$view-dashboard
ui/action$view-dashboard-in-background

Примеры настроек действий::

 <journal id="ecos-sync">
    <datasource>integrations/sync</datasource>
    <create>
        <variant title="Alfresco Records">
            <recordRef>integrations/sync@alfrecords</recordRef>
            <attribute name="type">alfrecords</attribute>
        </variant>
    </create>
    <actions>
        <action ref="ui/action$ecos-module-download" />
        <action ref="ui/action$delete" />
        <action ref="ui/action$edit" />
    </actions>
    <headers>
        <header key="module_id" default="true"/>
        <header key="name" default="true"/>
        <header key="type" default="true"/>
        <header key="syncDate" default="true"/>
        <header key="enabled" default="true"/>
    </headers>
 </journal>

Настройка действия, которое активно для записей с определенным mimetype контента::

 {
    "id": "edit-in-onlyoffice",
    "key": "edit.onlyoffice",
    "name": "Редактировать Документ",
    "type": "open-url", // тип действия должен соответствовать типу на UI
    "config": {
        "url": "/share/page/onlyoffice-edit?nodeRef=${recordRef}&new="
    },
    "evaluator": {
        "type": "predicate", // Тип evaluator'а для фильтрации действий
        "config": {
            "predicate": {
                "t": "in",
                "att": "_content.mimetype?str", // атрибут, который мы проверяем
                "val": [ //значения, на которые мы проверяем
                    "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                    "application/vnd.openxmlformats-officedocument.presentationml.presentation",
                    "text/plain",
                    "text/csv"
                ]
            }
        }
    }
 }

Данный конфиг достаточно положить в ecos-app/ui/action для микросервисов или в {alfresco_module_id}/src/main/resources/alfresco/module/{alfresco_module_id}/ui/action для Alfresco
