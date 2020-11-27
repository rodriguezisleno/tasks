# Mail components

## Abstract

We want to change the way we develop mails templates. In the past we had to handle a lot of logic that didn't belong to the mail itself. Our proposal is that the exercise of creating mail templates transforms into the exercise of composing mail components. In this document we will define how are we going to direct that transformation into a workflow that we can comprehend and implement.

## Introduction

Mail templates can be seen as a composition of two different kind of components: **Leaf** components and **container** components. The former ones are components that have no hierarchy and are self-contained, the latter ones are compositions of the former ones.

## Leaf components
As it was described before the leaf components are self-contained and have no hierarchy. Such components could be an image, a paragraph and so on. 

We have the following proposal for this kind of components:

```json
{
    "image": {
        "source": "https://url.to/image",
        "alt": "Alternative text"
    }
}
```

## Container components
The container components do have a hierarchy and are the ones that allow us to handle the case where a component has some part to be defined. Most of the components will be container components.

We have the following proposal for this kind of components:

```json
{
    "thumbnail": {
        "image": {
            "source": "https://url.to/image",
            "alt": "Alternative text",
            "height": 120,
            "width": 120
        },
        "hierarchy": "loud"
    }
}
```
```json
{
    "banner": {
        "thumbnail": {
            "image": {
                "source": "https://url.to/image",
                "alt": "Alternative text",
            },
            "badge": {
                "hierarchy": "loud",
                "type": "icon", 
                "feedback": "success",
                "text": ""
            }
        },
        "background_color": "#0000FF",
        "title": "Title",
        "subtitle": "Subtitle"
    }
}
```
```json
{
    "installments_section": {
        "overdue_date": "Vencen el 26 de enero",
        "badge": {
            "hierarchy": "loud",
            "type": "pill", 
            "feedback": "success",
            "text": "VENCIDO"
        },
        "installments_list": [
            {
                "image": {
                    "source": "https://url.to/image",
                    "alt": "Alternative text",
                },
                "title": "Cuota 3 de 9",
                "subtitle": "Spinner que brilla en la oscuridad",
                "price": {
                    "currency_symbol": "$",
                    "integer": 75,
                    "decimal": 44,
                    "thousand_separator": ".",
                    "decimal_separator": ",",
                    "type": "superscript"
                }
            }
        ]

    }
}
```

## Text

We will often have to handle content in these components. This content can acquire distinct forms as it does in native scenarios. We will have to support hypertext in the vast majority of the cases.

Some examples are: 

```json
{
    "text": "¿Tienes más dudas? Consulta nuestras <a href=\"http://link.to/faqs\">preguntas frecuentes.</a>"
}
```
```json
{
    "text": "Si ya pagaste no te preocupes, la acreditación del dinero puede tardar <strong>hasta 2 días hábiles</strong>."
}
```

### Translations
We don't want to handle translations (nor any kind of meaningful content) in the definition of the templates. So in the above example we expect that each text field will be filled with translated content.

### Assets (or just images)
The only assets we will be using are images. We have to different scenarios for the assets that we have to handle: 

- The needed asset has no variance on the data that is relevant to the template.
- The needed asset is impossible to know at the time of making the mail template. 

In the first scenario, the template makers will upload the asset and the asset will be hardcoded in the email template. In the second scenario, we can't know _a priori_ what the asset will be so we will expect for an URL to search for.

### Disambiguation
There's a case when we visually have the same exact component in a component that accepts childrens. In this case we propose to disambiguate by changing the key of the component and maintain the component structure.

Example
```json
{
    "card": {
        "text_1": "Some content",
        "text_2": "Some other content",
        "image_1": {
            "source": "http://link.to/image_1",
            "alt": "an image"
        },
        "image_2": {
            "source": "http://link.to/image_2",
            "alt": "another image"
        }
    } 
}
```

Where's the gain? 
- From backend you can reuse the way to build the component.
- From frontend we can univoquely identify the data that we need.

Where's the pain?
- Back & Front has to agree in what these keys are going to be.

## Components

### Header
```json
{
    "header": {
        "nickname": "GRODRIGUEZIS",
        "is_link_enabled": true
    }
}
```

### Button
```json
{
    "button": {
        "text": "Pagar",
        "hierarchy": "loud",
        "size": "large",
        "is_fullwidth": "true",
        "icon": "http://link.to/icon"
    }
}
```

### Thumbnail
```json
{
    "thumbnail": {
        "type": "image",
        "size": 32
    }
}
```

### Badge
```json
{
    "badge": {
        "hierarchy": "loud"
        "type": "icon",
        "feedback": "success",
        "text": "",
    }
}
```

### Card layout
```json
{
    "card_layout": {
        "banner": {
            "feedback": "information",
            "title": "Gonzalo, cerró el resumen de noviembre de tu tarjeta de Mercado Pago",
            "image": { --> Este puede venir del BE o definirse en el FE.
                "source": "http://link.to/image",
                "alt": "Tarjeta de Mercado Pago"
            }
        },
        "card": {
            "body": {
                "payment_summary": {
                    "supra_text": "Total a pagar",
                    "amount": "$ 2500",
                    "sub_text": "Pago mínimo $ 25"
                }
            },
            "recommendation": "Te recomendamos que lo hagas antes del 12 de diciembre, así evitas recargos.",
            "main_message": "Tú decides cómo quedar al día",
            "item_list": [
                {
                    "image": "http://link.to/image1",
                    "title": "Pago total",
                    "description": "No tendrás ningún recargo y recuperarás límite para usar."
                },
                {
                    "image": "http://link.to/image2",
                    "title": "Pago mínimo o parcial",
                    "description": "Pagas una parte ahora y el resto con intereses en el próximo resumen."
                },
                {
                    "image": "http://link.to/image3",
                    "title": "Cuotificar resumen",
                    "description": "Pagas la primera cuota ahora y las próximas en tus siguientes resúmenes."
                }
            ],
            "button": {
                "hierarchy": "loud",
                "text": "Pagar",
                "size": "large",
                "is_fullwidth": true,
            }
        }
        
    }
}
```

## Usage example

```js
module.exports = (context) => (
    <>
        <Header props={context.header}/>
        <CardLayout props={context.card_layout}>
            <CardBody>
                <AmountCard props={context.card_layout.card_body.payment_summary}/>
                <Hypertext props={context.card_layout.card_body.recommendation}/>
                <Hypertext props={context.card_layout.card_body.main_message}/>
                <ItemList props={context.card_layout.card_body.item_list}/>
                <Button props={context.card_layout.card_body.button}/>
                <Hypertext props={context.card_layout.card_body.disclaimer}/>
            </CardBody>
            <Body>
                <Hypertext props={context.card_layout.body.question}/>
                <Hypertext props={context.card_layout.body.team}/>
            </Body>
        <CardLayout/>
        <Footer props={context.footer}/>
    </>
);
    
```


## Preguntas
- ¿Cómo hacemos para manejar contextos diferentes de push, mails y whatsapp?
    - El contexto de todos los tipos de notificaciones son iguales.
    - Querer armar diferentes contextos para diferentes tipos de notificaciones implica mayor # de pegadas a la 
- Problemas con imágenes.
    - Tenemos que identificar los assets.
- ¿Podemos reutilizar el approach de armado dinámico?
    - En push notifications y whatsapp no se puede.
- ¿Cambiar el approach del contexto tiene valor?
    - TBD
- Journey de MP
    - ¿Cómo hacemos para resolver universo ML|MP o empujar Andes?
