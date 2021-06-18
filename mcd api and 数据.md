# https://mopstgfd.plexure.io

## /store/v3/categories?dayPart=DAY_MENU

### 获取 store 分类

```json
[{
	"id": 30,
	"name": "Value Sets",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/30",
	"timeOfDay": null,
	"promoted": false,
	"displayOrder": 2
}, {
	"id": 2,
	"name": "Burgers",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/2",
	"timeOfDay": null,
	"promoted": false,
	"displayOrder": 3
}, {
	"id": 3,
	"name": "Sides",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/3",
	"timeOfDay": null,
	"promoted": false,
	"displayOrder": null
}, {
	"id": 4,
	"name": "Sweets",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/4",
	"timeOfDay": null,
	"promoted": false,
	"displayOrder": null
}, {
	"id": 5,
	"name": "Beverages",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/5",
	"timeOfDay": null,
	"promoted": false,
	"displayOrder": null
}, {
	"id": 6,
	"name": "Happy Sets",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/6",
	"timeOfDay": null,
	"promoted": false,
	"displayOrder": null
}, {
	"id": 20,
	"name": "Value Lunch",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/20",
	"timeOfDay": {
		"startTime": "10:30:00",
		"endTime": "14:00:00"
	},
	"promoted": false,
	"displayOrder": null
}, {
	"id": 21,
	"name": "Yoru Mac",
	"comment": null,
	"image": "https://mopstgfd.plexure.io/category-images/category_mobile/21",
	"timeOfDay": {
		"startTime": "17:00:00",
		"endTime": "00:00:00"
	},
	"promoted": false,
	"displayOrder": null
}]
```



## /store/v3/stores/1620/products?dayPart=DAY_MENU&categoryId=3&includeInOutage=true

### 获取 store 特定分类下的商品

```json
[{
	"productCode": 2010,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/2010"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 150.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 150.0
	}, {
		"priceCode": "OTHER",
		"price": 200.0
	}],
	"localisedProductName": {
		"menuName": "McFry (S)",
		"variantName": "McFry (S)",
		"cartName": "McFry (S)"
	},
	"dayPart": "DAY_MENU",
	"startDateTime": "2012-01-01T00:00:00Z",
	"endDateTime": "2099-01-01T00:00:00Z",
	"displayOrder": 2990,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 2020,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/2020"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 280.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 280.0
	}, {
		"priceCode": "OTHER",
		"price": 320.0
	}],
	"localisedProductName": {
		"menuName": "McFry (M)",
		"variantName": "McFry (M)",
		"cartName": "McFry (M)"
	},
	"dayPart": "DAY_MENU",
	"startDateTime": "2015-01-05T05:00:00Z",
	"endDateTime": "2099-01-01T05:00:00Z",
	"displayOrder": 3000,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 2050,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/2050"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 330.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 330.0
	}, {
		"priceCode": "OTHER",
		"price": 380.0
	}],
	"localisedProductName": {
		"menuName": "McFry (L)",
		"variantName": "McFry (L)",
		"cartName": "McFry (L)"
	},
	"dayPart": "DAY_MENU",
	"startDateTime": "2015-01-05T05:00:00Z",
	"endDateTime": "2099-01-05T05:00:00Z",
	"displayOrder": 3010,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 1610,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/1610"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 200.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 200.0
	}, {
		"priceCode": "OTHER",
		"price": 240.0
	}],
	"localisedProductName": {
		"menuName": "Chicken McNuggets 8Pcs",
		"variantName": "Chicken McNuggets 7Pcs",
		"cartName": "Chicken McNuggets 6Pcs"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2012-01-01T00:00:00Z",
	"endDateTime": "2099-01-01T00:00:00Z",
	"displayOrder": 3020,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [{
		"choiceProductCode": 7251,
		"defaultProduct": {
			"productCode": 6048,
			"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6048"],
			"categoryId": null,
			"demotedItem": false,
			"productClass": "NON_FOOD_PRODUCT",
			"inOutage": false,
			"status": "ACTIVE",
			"priceList": [{
				"priceCode": "EATIN",
				"price": 0.0
			}, {
				"priceCode": "TAKEOUT",
				"price": 0.0
			}, {
				"priceCode": "OTHER",
				"price": 0.0
			}],
			"localisedProductName": {
				"menuName": "BBQ Sauce 1",
				"variantName": "BBQ Sauce 2",
				"cartName": "BBQ Sauce 5"
			},
			"dayPart": "BREAKFAST_DAY_MENU",
			"startDateTime": "2011-12-31T15:00:00Z",
			"endDateTime": "2098-12-31T15:00:00Z",
			"displayOrder": null,
			"calculatedTotal": null,
			"productOptions": [{
				"productCode": 6048,
				"localisedProductName": {
					"menuName": "BBQ Sauce 1",
					"variantName": "BBQ Sauce 2",
					"cartName": "BBQ Sauce 5"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 6049,
				"localisedProductName": {
					"menuName": "Mustard Sauce",
					"variantName": "Mustard Sauce",
					"cartName": "Mustard Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 708001,
				"localisedProductName": {
					"menuName": "No Sauce",
					"variantName": "No Sauce",
					"cartName": "No Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}],
			"localisedComment": null,
			"additionalChoices": null,
			"daysOfWeek": null,
			"timeOfDay": null,
			"priceDifference": null,
			"hasProductOptions": true,
			"recommended": false,
			"generic": false
		},
		"localisedChoiceName": null,
		"isHidden": false,
		"notPartOfOriginalCombo": false,
		"preselectDefaultProduct": false
	}],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 1670,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/1670"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 580.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 580.0
	}, {
		"priceCode": "OTHER",
		"price": 680.0
	}],
	"localisedProductName": {
		"menuName": "Chicken McNuggets 15Pcs",
		"variantName": "Chicken McNuggets 15Pcs",
		"cartName": "Chicken McNuggets 15Pcs"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2014-04-01T05:00:00Z",
	"endDateTime": "2099-04-01T05:00:00Z",
	"displayOrder": 3022,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [{
		"choiceProductCode": 7251,
		"defaultProduct": {
			"productCode": 6048,
			"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6048"],
			"categoryId": null,
			"demotedItem": false,
			"productClass": "NON_FOOD_PRODUCT",
			"inOutage": false,
			"status": "ACTIVE",
			"priceList": [{
				"priceCode": "EATIN",
				"price": 0.0
			}, {
				"priceCode": "TAKEOUT",
				"price": 0.0
			}, {
				"priceCode": "OTHER",
				"price": 0.0
			}],
			"localisedProductName": {
				"menuName": "BBQ Sauce 1",
				"variantName": "BBQ Sauce 2",
				"cartName": "BBQ Sauce 5"
			},
			"dayPart": "BREAKFAST_DAY_MENU",
			"startDateTime": "2011-12-31T15:00:00Z",
			"endDateTime": "2098-12-31T15:00:00Z",
			"displayOrder": null,
			"calculatedTotal": null,
			"productOptions": [{
				"productCode": 6048,
				"localisedProductName": {
					"menuName": "BBQ Sauce 1",
					"variantName": "BBQ Sauce 2",
					"cartName": "BBQ Sauce 5"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 6049,
				"localisedProductName": {
					"menuName": "Mustard Sauce",
					"variantName": "Mustard Sauce",
					"cartName": "Mustard Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 708001,
				"localisedProductName": {
					"menuName": "No Sauce",
					"variantName": "No Sauce",
					"cartName": "No Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}],
			"localisedComment": null,
			"additionalChoices": null,
			"daysOfWeek": null,
			"timeOfDay": null,
			"priceDifference": null,
			"hasProductOptions": true,
			"recommended": false,
			"generic": false
		},
		"localisedChoiceName": null,
		"isHidden": false,
		"notPartOfOriginalCombo": false,
		"preselectDefaultProduct": false
	}, {
		"choiceProductCode": 7251,
		"defaultProduct": {
			"productCode": 6048,
			"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6048"],
			"categoryId": null,
			"demotedItem": false,
			"productClass": "NON_FOOD_PRODUCT",
			"inOutage": false,
			"status": "ACTIVE",
			"priceList": [{
				"priceCode": "EATIN",
				"price": 0.0
			}, {
				"priceCode": "TAKEOUT",
				"price": 0.0
			}, {
				"priceCode": "OTHER",
				"price": 0.0
			}],
			"localisedProductName": {
				"menuName": "BBQ Sauce 1",
				"variantName": "BBQ Sauce 2",
				"cartName": "BBQ Sauce 5"
			},
			"dayPart": "BREAKFAST_DAY_MENU",
			"startDateTime": "2011-12-31T15:00:00Z",
			"endDateTime": "2098-12-31T15:00:00Z",
			"displayOrder": null,
			"calculatedTotal": null,
			"productOptions": [{
				"productCode": 6048,
				"localisedProductName": {
					"menuName": "BBQ Sauce 1",
					"variantName": "BBQ Sauce 2",
					"cartName": "BBQ Sauce 5"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 6049,
				"localisedProductName": {
					"menuName": "Mustard Sauce",
					"variantName": "Mustard Sauce",
					"cartName": "Mustard Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 708001,
				"localisedProductName": {
					"menuName": "No Sauce",
					"variantName": "No Sauce",
					"cartName": "No Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}],
			"localisedComment": null,
			"additionalChoices": null,
			"daysOfWeek": null,
			"timeOfDay": null,
			"priceDifference": null,
			"hasProductOptions": true,
			"recommended": false,
			"generic": false
		},
		"localisedChoiceName": null,
		"isHidden": false,
		"notPartOfOriginalCombo": false,
		"preselectDefaultProduct": false
	}, {
		"choiceProductCode": 7251,
		"defaultProduct": {
			"productCode": 6048,
			"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6048"],
			"categoryId": null,
			"demotedItem": false,
			"productClass": "NON_FOOD_PRODUCT",
			"inOutage": false,
			"status": "ACTIVE",
			"priceList": [{
				"priceCode": "EATIN",
				"price": 0.0
			}, {
				"priceCode": "TAKEOUT",
				"price": 0.0
			}, {
				"priceCode": "OTHER",
				"price": 0.0
			}],
			"localisedProductName": {
				"menuName": "BBQ Sauce 1",
				"variantName": "BBQ Sauce 2",
				"cartName": "BBQ Sauce 5"
			},
			"dayPart": "BREAKFAST_DAY_MENU",
			"startDateTime": "2011-12-31T15:00:00Z",
			"endDateTime": "2098-12-31T15:00:00Z",
			"displayOrder": null,
			"calculatedTotal": null,
			"productOptions": [{
				"productCode": 6048,
				"localisedProductName": {
					"menuName": "BBQ Sauce 1",
					"variantName": "BBQ Sauce 2",
					"cartName": "BBQ Sauce 5"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 6049,
				"localisedProductName": {
					"menuName": "Mustard Sauce",
					"variantName": "Mustard Sauce",
					"cartName": "Mustard Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}, {
				"productCode": 708001,
				"localisedProductName": {
					"menuName": "No Sauce",
					"variantName": "No Sauce",
					"cartName": "No Sauce"
				},
				"optionType": "Sauce",
				"dayPart": "BREAKFAST_DAY_MENU",
				"inOutage": false
			}],
			"localisedComment": null,
			"additionalChoices": null,
			"daysOfWeek": null,
			"timeOfDay": null,
			"priceDifference": null,
			"hasProductOptions": true,
			"recommended": false,
			"generic": false
		},
		"localisedChoiceName": null,
		"isHidden": false,
		"notPartOfOriginalCombo": false,
		"preselectDefaultProduct": false
	}],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 2080,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/2080"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 150.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 150.0
	}, {
		"priceCode": "OTHER",
		"price": 170.0
	}],
	"localisedProductName": {
		"menuName": "Shaka-Chicki",
		"variantName": "Shaka-Chicki",
		"cartName": "Shaka-Chicki"
	},
	"dayPart": "DAY_MENU",
	"startDateTime": "2014-01-20T05:00:00Z",
	"endDateTime": "2099-12-31T23:59:00Z",
	"displayOrder": 3030,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [{
		"choiceProductCode": 9997021,
		"defaultProduct": {
			"productCode": 6042,
			"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6042"],
			"categoryId": null,
			"demotedItem": false,
			"productClass": "NON_FOOD_PRODUCT",
			"inOutage": false,
			"status": "ACTIVE",
			"priceList": [{
				"priceCode": "EATIN",
				"price": 0.0
			}, {
				"priceCode": "TAKEOUT",
				"price": 0.0
			}, {
				"priceCode": "OTHER",
				"price": 0.0
			}],
			"localisedProductName": {
				"menuName": "Shaka-Chicki Cheddar Cheese",
				"variantName": "Shaka-Chicki Cheddar Cheese",
				"cartName": "Shaka-Chicki Cheddar Cheese"
			},
			"dayPart": "BREAKFAST_DAY_MENU",
			"startDateTime": "2011-12-31T15:00:00Z",
			"endDateTime": "2098-12-31T15:00:00Z",
			"displayOrder": null,
			"calculatedTotal": null,
			"productOptions": [],
			"localisedComment": null,
			"additionalChoices": null,
			"daysOfWeek": null,
			"timeOfDay": null,
			"priceDifference": null,
			"hasProductOptions": false,
			"recommended": false,
			"generic": false
		},
		"localisedChoiceName": null,
		"isHidden": false,
		"notPartOfOriginalCombo": false,
		"preselectDefaultProduct": false
	}],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 2604,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/2604"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 230.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 230.0
	}, {
		"priceCode": "OTHER",
		"price": 270.0
	}],
	"localisedProductName": {
		"menuName": "Sweet Corn",
		"variantName": "Sweet Corn",
		"cartName": "Sweet Corn"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2012-01-01T00:00:00Z",
	"endDateTime": "2099-01-01T00:00:00Z",
	"displayOrder": 3070,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}, {
	"productCode": 2310,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/2310"],
	"categoryId": 3,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 280.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 280.0
	}, {
		"priceCode": "OTHER",
		"price": 340.0
	}],
	"localisedProductName": {
		"menuName": "Side Salad",
		"variantName": "A la carte",
		"cartName": "Side Salad"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2015-05-25T05:00:00Z",
	"endDateTime": "2099-05-25T05:00:00Z",
	"displayOrder": 5160,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [{
		"choiceProductCode": 9997136,
		"defaultProduct": {
			"productCode": 6357,
			"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6357"],
			"categoryId": null,
			"demotedItem": false,
			"productClass": "NON_FOOD_PRODUCT",
			"inOutage": false,
			"status": "ACTIVE",
			"priceList": [{
				"priceCode": "EATIN",
				"price": 0.0
			}, {
				"priceCode": "TAKEOUT",
				"price": 0.0
			}, {
				"priceCode": "OTHER",
				"price": 0.0
			}],
			"localisedProductName": {
				"menuName": "Sesame Dressing",
				"variantName": "Sesame Dressing",
				"cartName": "Sesame Dressing"
			},
			"dayPart": "BREAKFAST_DAY_MENU",
			"startDateTime": "2011-12-31T15:00:00Z",
			"endDateTime": "2098-12-31T15:00:00Z",
			"displayOrder": null,
			"calculatedTotal": null,
			"productOptions": [],
			"localisedComment": null,
			"additionalChoices": null,
			"daysOfWeek": null,
			"timeOfDay": null,
			"priceDifference": null,
			"hasProductOptions": false,
			"recommended": false,
			"generic": false
		},
		"localisedChoiceName": null,
		"isHidden": false,
		"notPartOfOriginalCombo": false,
		"preselectDefaultProduct": false
	}],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": null,
	"hasProductOptions": false,
	"recommended": false,
	"generic": false
}]
```



## store/v3/stores/1620/choices/1610/7251?dayPart=DAY_MENU&includeInOutage=true

### 获取 store（1620） 下特定产品 （1610）的额外选项的完整信息（additionalChoices.choiceProductCode=7251）

```json
[{
	"productCode": 708001,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/708001"],
	"categoryId": 10,
	"demotedItem": false,
	"productClass": "PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 0.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 0.0
	}, {
		"priceCode": "OTHER",
		"price": 0.0
	}],
	"localisedProductName": {
		"menuName": "No Sauce",
		"variantName": "No Sauce",
		"cartName": "No Sauce"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2012-01-01T00:00:00",
	"endDateTime": "2099-12-31T00:00:00",
	"displayOrder": 99,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": [{
		"priceCode": "EATIN",
		"price": 0.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 0.0
	}, {
		"priceCode": "OTHER",
		"price": 0.0
	}],
	"hasProductOptions": true,
	"recommended": false,
	"generic": false
}, {
	"productCode": 6048,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6048"],
	"categoryId": null,
	"demotedItem": false,
	"productClass": "NON_FOOD_PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 0.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 0.0
	}, {
		"priceCode": "OTHER",
		"price": 0.0
	}],
	"localisedProductName": {
		"menuName": "BBQ Sauce 1",
		"variantName": "BBQ Sauce 2",
		"cartName": "BBQ Sauce 5"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2012-01-01T00:00:00",
	"endDateTime": "2099-01-01T00:00:00",
	"displayOrder": null,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": [{
		"priceCode": "EATIN",
		"price": 0.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 0.0
	}, {
		"priceCode": "OTHER",
		"price": 0.0
	}],
	"hasProductOptions": true,
	"recommended": false,
	"generic": false
}, {
	"productCode": 6049,
	"productImages": ["https://mopstgfd.plexure.io/product-images/product_mobile/6049"],
	"categoryId": null,
	"demotedItem": false,
	"productClass": "NON_FOOD_PRODUCT",
	"inOutage": false,
	"status": "ACTIVE",
	"priceList": [{
		"priceCode": "EATIN",
		"price": 0.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 0.0
	}, {
		"priceCode": "OTHER",
		"price": 0.0
	}],
	"localisedProductName": {
		"menuName": "Mustard Sauce",
		"variantName": "Mustard Sauce",
		"cartName": "Mustard Sauce"
	},
	"dayPart": "BREAKFAST_DAY_MENU",
	"startDateTime": "2012-01-01T00:00:00",
	"endDateTime": "2099-01-01T00:00:00",
	"displayOrder": null,
	"calculatedTotal": null,
	"productOptions": [],
	"localisedComment": null,
	"additionalChoices": [],
	"daysOfWeek": null,
	"timeOfDay": null,
	"priceDifference": [{
		"priceCode": "EATIN",
		"price": 0.0
	}, {
		"priceCode": "TAKEOUT",
		"price": 0.0
	}, {
		"priceCode": "OTHER",
		"price": 0.0
	}],
	"hasProductOptions": true,
	"recommended": false,
	"generic": false
}]
```



## GET /order/v3/payments HTTP/1.1

### 获取支付方式

```
[]
```



## POST https://mopfd.plexure.io/order/v3/orders

### 创建订单

```json
{
	"appBundleId": "jp.co.mcdonalds.android",
	"appVersion": "5.1.61(206)(50161)",
	"customerDetails": {
		"emailAddress": "john@theplant.jp",
		"firstName": "",
		"lastName": ""
	},
	"items": [{
		"offerIsRepeatable": false,
		"items": [{
			"offerIsRepeatable": false,
			"productCode": 6049,
			"productName": "Mustard Sauce",
			"quantity": 1,
			"shouldBeBaseItem": true,
			"unitPrice": 0.0
		}, {
			"offerIsRepeatable": false,
			"productCode": 6049,
			"productName": "Mustard Sauce",
			"quantity": 1,
			"shouldBeBaseItem": true,
			"unitPrice": 0.0
		}],
		"productCode": 1670,
		"productName": "Chicken McNuggets 15Pcs",
		"quantity": 1,
		"shouldBeBaseItem": false,
		"unitPrice": 580.0
	}],
	"dayPart": "DAY",
	"orderType": "EATIN",
	"platform": "android",
	"storeId": 13157,
	"storeName": "池袋北口店"
}

// response: 
{"orderFullId":"2021020203363301315770711","orderId":"70711","consumerId":"500e6640-8b14-4644-8310-3621f88453ee","totalAmount":580.0,"dateCreated":"2021-02-02T03:36:32.8980471Z","dateUpdated":"2021-02-02T03:36:33.0352759Z","dateCheckedIn":null,"storeId":13157,"storeName":"池袋北口店","appVersion":"5.1.61(206)(50161)","platform":"android","appBundleId":"jp.co.mcdonalds.android","status":"Pending","items":[{"productCode":1670,"productName":"Chicken McNuggets 15Pcs","quantity":1,"unitPrice":580.0,"grill":null,"items":[{"productCode":6049,"productName":"Mustard Sauce","quantity":1,"unitPrice":0.0,"grill":null,"items":null,"offerId":null,"offerInstanceUniqueId":null,"shouldBeBaseItem":true,"taxRate":1.1,"taxAmount":0.0},{"productCode":6049,"productName":"Mustard Sauce","quantity":1,"unitPrice":0.0,"grill":null,"items":null,"offerId":null,"offerInstanceUniqueId":null,"shouldBeBaseItem":true,"taxRate":1.1,"taxAmount":0.0}],"offerId":null,"offerInstanceUniqueId":null,"shouldBeBaseItem":false,"taxRate":1.1,"taxAmount":52.73}],"orderType":"EatIn","dayPart":"Day","displayOrderNumber":"M89","orderTransaction":null,"customerDetails":{"firstName":"","lastName":"","emailAddress":"john@theplant.jp","language":"en"},"businessDate":null,"pickupType":"FrontCounter","tableNumber":null,"comment":null,"parkingLotNumber":null}
```



## PUT https://mopfd.plexure.io/order/v3/orders/{orderId}/cancel

### 取消订单

```
"status":"Canceled"
```



## POST https://mopfd.plexure.io/order/v3/paypay/2021020203363301315770711/authorize?errorUrl=https://mopfd.plexure.io/order/v3/redirect/cancel?appUrlScheme=mcdonaldsjp&cancelUrl=https://mopfd.plexure.io/order/v3/redirect/error?appUrlScheme=mcdonaldsjp&successUrl=https://mopfd.plexure.io/order/v3/redirect/success?appUrlScheme=mcdonaldsjp&linePayUrlScheme=true

### 为订单授权支付方式

返回值也许是 html 网页也许是 url



## PUT https://mopstgfd.plexure.io/order/v3/paypay/20210202035413100000362997/checkin?pickupType=TableService&orderType=EATIN&tableNumber=444

```json
RES:

{
    "orderFullId":"20210202035413100000362997",
    "orderId":"62997",
    "consumerId":"4ba952d8-f4ff-49e8-b32f-3621de19b183",
    "totalAmount":580.0,
    "dateCreated":"2021-02-02T03:54:13.3859902Z",
    "dateUpdated":"2021-02-02T03:54:57.9830589Z",
    "dateCheckedIn":"2021-02-02T03:54:57.7617272Z",
    "storeId":1000003,
    "storeName":"Mock 1000003",
    "appVersion":"5.1.61(206)(50161)",
    "platform":"android",
    "appBundleId":"jp.co.mcdonalds.android.staging",
    "status":"FailedCapture",
    "items":[
        {
            "productCode":1670,
            "productName":"Chicken McNuggets 15Pcs",
            "quantity":1,
            "unitPrice":580.0,
            "grill":[
                
            ],
            "items":[
                {
                    "productCode":6049,
                    "productName":"Mustard Sauce",
                    "quantity":1,
                    "unitPrice":0.0,
                    "grill":[
                        
                    ],
                    "items":[
                        
                    ],
                    "offerId":null,
                    "offerInstanceUniqueId":null,
                    "shouldBeBaseItem":true,
                    "taxRate":1.1,
                    "taxAmount":0.0
                },
                {
                    "productCode":6049,
                    "productName":"Mustard Sauce",
                    "quantity":1,
                    "unitPrice":0.0,
                    "grill":[
                        
                    ],
                    "items":[
                        
                    ],
                    "offerId":null,
                    "offerInstanceUniqueId":null,
                    "shouldBeBaseItem":true,
                    "taxRate":1.1,
                    "taxAmount":0.0
                }
            ],
            "offerId":null,
            "offerInstanceUniqueId":null,
            "shouldBeBaseItem":false,
            "taxRate":1.1,
            "taxAmount":52.73
        }
    ],
    "orderType":"EatIn",
    "dayPart":"Day",
    "displayOrderNumber":"T444",
    "orderTransaction":{
        "authCode":null,
        "accountId":null,
        "cardId":null,
        "cardNumber":null,
        "paymentType":"ExternalProvider",
        "externalProvider":"paypay",
        "externalProviderRedirectWebUrl":"<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">\r\n<HTML>\r\n<HEAD>\r\n<META http-equiv=\"Cache-Control\" content=\"no-store, no-cache\">\r\n<META http-equiv=\"Pragma\" content=\"no-cache\">\r\n<META http-equiv=\"Expires\" content=\"0\">\r\n<META http-equiv=\"Content-Type\" content=\"text/html; charset=Shift_JIS\" />\r\n</HEAD>\r\n<SCRIPT language=\"javascript\" type=\"text/javascript\">\r\n<!--\r\nfunction OnLoadEvent(){\r\n document.getElementById(\"continue\").disabled = true;\r\n document.getElementById(\"mainForm\").submit();\r\n}\r\n// -->\r\n</SCRIPT>\r\n<BODY onLoad=\"OnLoadEvent();\">\r\n<FORM id=\"mainForm\" name=\"mainForm\" action=\"https://api.veritrans.co.jp/dummy-paypay/paypay?csk=VgZZoqUkw8lyl30gU3vws0mxHfFb0ll9pdJHcPc4Knpai8P8cAs8Qvykl51iwwf6\" method=\"POST\" onSubmit=\"document.getElementById('continue').disabled=true; return true;\">\r\n<BR><BR><CENTER>\r\n<H4>PayPayの決済ページに移動します</H4>\r\n<INPUT type=\"submit\" id=\"continue\" name=\"continue\" value=\"次へ\">\r\n</CENTER>\r\n<INPUT type=\"hidden\" name=\"pay_method\" value=\"paypay\">\r\n<INPUT type=\"hidden\" name=\"merchant_id\" value=\"30132\">\r\n<INPUT type=\"hidden\" name=\"service_id\" value=\"101\">\r\n<INPUT type=\"hidden\" name=\"cust_code\" value=\"30132-101\">\r\n<INPUT type=\"hidden\" name=\"sps_cust_no\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"sps_payment_no\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"order_id\" value=\"PAYPAY-prd-00000000006589090\">\r\n<INPUT type=\"hidden\" name=\"item_id\" value=\"20210202035413100000362997\">\r\n<INPUT type=\"hidden\" name=\"pay_item_id\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"item_name\" value=\"McDonald's\">\r\n<INPUT type=\"hidden\" name=\"tax\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"amount\" value=\"580\">\r\n<INPUT type=\"hidden\" name=\"pay_type\" value=\"0\">\r\n<INPUT type=\"hidden\" name=\"auto_charge_type\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"service_type\" value=\"0\">\r\n<INPUT type=\"hidden\" name=\"div_settele\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"last_charge_month\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"camp_type\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"tracking_id\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"terminal_type\" value=\"0\">\r\n<INPUT type=\"hidden\" name=\"success_url\" value=\"https://api.veritrans.co.jp/tercerog/webinterface/GWTripartiteNACommandRcv/paypay/ResultRedirect/online?pypy_mc_order_id=PAYPAY-prd-00000000006589090&amp;last_user_action=10&amp;check_sum=cc8440d9b076f94da65ce1b626933c1a774c2421\">\r\n<INPUT type=\"hidden\" name=\"cancel_url\" value=\"https://api.veritrans.co.jp/tercerog/webinterface/GWTripartiteNACommandRcv/paypay/ResultRedirect/online?pypy_mc_order_id=PAYPAY-prd-00000000006589090&amp;last_user_action=20&amp;check_sum=e434b8b980b1969f9f17a8e918fbe00cc57ff426\">\r\n<INPUT type=\"hidden\" name=\"error_url\" value=\"https://api.veritrans.co.jp/tercerog/webinterface/GWTripartiteNACommandRcv/paypay/ResultRedirect/online?pypy_mc_order_id=PAYPAY-prd-00000000006589090&amp;last_user_action=30&amp;check_sum=a2ecd64d839893422aa3fb1505f0dcc9d21d7bd2\">\r\n<INPUT type=\"hidden\" name=\"pagecon_url\" value=\"http://127.0.0.1/tercerog/webinterface/GWTripartiteNASJISCommandRcv/paypay/AuthorizeNotify/online\">\r\n<INPUT type=\"hidden\" name=\"free1\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"free2\" value=\"01069951\">\r\n<INPUT type=\"hidden\" name=\"free3\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"free_csv\" value=\"\">\r\n<INPUT type=\"hidden\" name=\"request_date\" value=\"20210202125436\">\r\n<INPUT type=\"hidden\" name=\"limit_second\" value=\"600\">\r\n<INPUT type=\"hidden\" name=\"sps_hashcode\" value=\"63b0bebf21bb2001690e3f8c9db640c1706f89e2\">\r\n</FORM>\r\n</BODY>\r\n</HTML>\r\n"
    },
    "customerDetails":{
        "firstName":"",
        "lastName":"",
        "emailAddress":"john@theplant.jp",
        "language":"en"
    },
    "businessDate":"2021-02-02T00:00:00",
    "pickupType":"TableService",
    "tableNumber":444,
    "comment":null,
    "parkingLotNumber":null
}
```





## GET /store/v3/stores/13157 HTTP/1.1

### 获取门店信息 {storeId}

```json
{"id":13157,"isEnabled":true,"isOnline":true,"name":"池袋北口店","address":"東京都豊島区東池袋１－４１－６","phone":"03-3971-2239","email":null,"latitude":35.73151838193434,"longitude":139.7128392998659,"city":null,"zipCode":"1700013","memo":"臨時営業時間 | お持ち帰りのみの営業 20:00～23:00","distributions":[],"openingHours":[{"date":"2021-02-02T00:00:00","day":"Tuesday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null},{"date":"2021-02-03T00:00:00","day":"Wednesday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null},{"date":"2021-02-04T00:00:00","day":"Thursday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null},{"date":"2021-02-05T00:00:00","day":"Friday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null},{"date":"2021-02-06T00:00:00","day":"Saturday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null},{"date":"2021-02-07T00:00:00","day":"Sunday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null},{"date":"2021-02-08T00:00:00","day":"Monday","breakfastMenuHours":{"startTime":"07:00:00","endTime":"10:30:00"},"dayMenuHours":{"startTime":"10:30:00","endTime":"23:00:00"},"tableDeliveryHours":{"startTime":"07:00:00","endTime":"19:30:00"},"curbSideDeliveryHours":{"startTime":"07:00:00","endTime":"22:00:00"},"comment":null}],"additionalProperties":{"numSeats":137,"numCarparks":0},"features":[],"featureList":["FREE_WIFI","BF","TABLE_DELIVERY","FRONT_COUNTER_EAT_IN"]}
```



# store 获取优惠券

某个 store 获取使用优惠券产品（通过 offer.id, code, name），获取到的数据结构：

- productCode：当前优惠券产品的code
- baseProducts 主产品节点（无选项）
- choices 选择性产品节点（多选项）
  - choiceProductCode 可选择产品的 code，用于获取所有选项的参数
  - defaultProduct 默认产品

```json
{
  "baseProducts": [
    {
      "categoryId": 3,
      "productCode": 1610,
      "displayOrder": 5140,
      "productImages": [
        "https://mopfd.plexure.io/product-images/product_mobile/1610"
      ],
      "inOutage": false,
      "recommended": false,
      "localisedProductName": {
        "cartName": "Chicken McNuggets 5Pcs",
        "menuName": "Chicken McNuggets 5Pcs",
        "variantName": "Chicken McNuggets 5Pcs"
      },
      "dayPart": "BREAKFAST_DAY_MENU",
      "productOptions": [],
      "priceList": [
        {
          "price": 100,
          "priceCode": "EATIN"
        },
        {
          "price": 100,
          "priceCode": "TAKEOUT"
        },
        {
          "price": 100,
          "priceCode": "OTHER"
        }
      ],
      "productClass": "PRODUCT",
      "status": "ACTIVE"
    },
    {
      "categoryId": 3,
      "productCode": 2010,
      "displayOrder": 5110,
      "productImages": [
        "https://mopfd.plexure.io/product-images/product_mobile/2010"
      ],
      "inOutage": false,
      "recommended": false,
      "localisedProductName": {
        "cartName": "McFry (S)",
        "menuName": "McFry (S)",
        "variantName": "McFry (S)"
      },
      "dayPart": "DAY_MENU",
      "productOptions": [],
      "priceList": [
        {
          "price": 100,
          "priceCode": "EATIN"
        },
        {
          "price": 100,
          "priceCode": "TAKEOUT"
        },
        {
          "price": 100,
          "priceCode": "OTHER"
        }
      ],
      "productClass": "PRODUCT",
      "status": "ACTIVE"
    },
    {
      "categoryId": 0,
      "productCode": 8040,
      "displayOrder": 0,
      "productImages": [
        "https://mopfd.plexure.io/product-images/product_mobile/8040"
      ],
      "inOutage": false,
      "recommended": false,
      "localisedProductName": {
        "cartName": "Ronald McDonald's House Donation",
        "menuName": "Ronald McDonald's House Donation",
        "variantName": "Ronald McDonald's House Donation"
      },
      "dayPart": "BREAKFAST_DAY_MENU",
      "productOptions": [],
      "priceList": [
        {
          "price": 10,
          "priceCode": "EATIN"
        },
        {
          "price": 10,
          "priceCode": "TAKEOUT"
        },
        {
          "price": 10,
          "priceCode": "OTHER"
        }
      ],
      "productClass": "COUPONS",
      "status": "ACTIVE"
    }
  ],
  "calculatedComboPrice": [
    {
      "price": 410,
      "priceCode": "EATIN"
    },
    {
      "price": 410,
      "priceCode": "OTHER"
    },
    {
      "price": 410,
      "priceCode": "TAKEOUT"
    }
  ],
  "choices": [
    {
      "choiceProductCode": 7251,
      "defaultProduct": {
        "categoryId": 3,
        "productCode": 6020,
        "displayOrder": 0,
        "productImages": [
          "https://mopfd.plexure.io/product-images/product_mobile/6020"
        ],
        "inOutage": false,
        "recommended": false,
        "localisedProductName": {
          "cartName": "Snow Crab & Spiny Lobster Sauce",
          "menuName": "Snow Crab & Spiny Lobster Sauce",
          "variantName": "A la carte"
        },
        "dayPart": "BREAKFAST_DAY_MENU",
        "productOptions": [],
        "priceList": [
          {
            "price": 0,
            "priceCode": "EATIN"
          },
          {
            "price": 0,
            "priceCode": "TAKEOUT"
          },
          {
            "price": 0,
            "priceCode": "OTHER"
          }
        ],
        "productClass": "NON_FOOD_PRODUCT",
        "status": "ACTIVE"
      },
      "isHidden": false,
      "preselectDefaultProduct": false,
      "notPartOfOriginalCombo": true
    },
    {
      "choiceProductCode": 9997007,
      "defaultProduct": {
        "categoryId": 5,
        "productCode": 3110,
        "displayOrder": 6130,
        "productImages": [
          "https://mopfd.plexure.io/product-images/product_mobile/3110"
        ],
        "inOutage": false,
        "recommended": false,
        "localisedProductName": {
          "cartName": "Coca Cola (S)",
          "menuName": "Coca Cola",
          "variantName": "S size"
        },
        "dayPart": "BREAKFAST_DAY_MENU",
        "productOptions": [
          {
            "localisedProductName": {
              "cartName": "Coca Cola (S)",
              "menuName": "Coca Cola",
              "variantName": "S size"
            },
            "dayPart": "BREAKFAST_DAY_MENU",
            "optionType": "AlternateSize",
            "productCode": 3110
          },
          {
            "localisedProductName": {
              "cartName": "Coca Cola (M)",
              "menuName": "Coca Cola",
              "variantName": "M size"
            },
            "dayPart": "BREAKFAST_DAY_MENU",
            "optionType": "AlternateSize",
            "productCode": 3120
          },
          {
            "localisedProductName": {
              "cartName": "Coca Cola (L)",
              "menuName": "Coca Cola",
              "variantName": "L size"
            },
            "dayPart": "BREAKFAST_DAY_MENU",
            "optionType": "AlternateSize",
            "productCode": 3150
          }
        ],
        "priceList": [
          {
            "price": 103,
            "priceCode": "EATIN"
          },
          {
            "price": 103,
            "priceCode": "TAKEOUT"
          },
          {
            "price": 103,
            "priceCode": "OTHER"
          }
        ],
        "productClass": "PRODUCT",
        "status": "ACTIVE"
      },
      "isHidden": false,
      "preselectDefaultProduct": false,
      "notPartOfOriginalCombo": false
    },
    {
      "choiceProductCode": 9997008,
      "defaultProduct": {
        "categoryId": 43,
        "productCode": 6671,
        "displayOrder": 0,
        "productImages": [
          "https://mopfd.plexure.io/product-images/product_mobile/6671"
        ],
        "inOutage": false,
        "recommended": false,
        "localisedProductName": {
          "cartName": "Story book Kabocha soup no ofuro",
          "menuName": "Story book Kabocha soup no ofuro",
          "variantName": "A la carte"
        },
        "dayPart": "BREAKFAST_DAY_MENU",
        "productOptions": [],
        "priceList": [
          {
            "price": 97,
            "priceCode": "EATIN"
          },
          {
            "price": 97,
            "priceCode": "TAKEOUT"
          },
          {
            "price": 97,
            "priceCode": "OTHER"
          }
        ],
        "productClass": "NON_FOOD_PRODUCT",
        "status": "ACTIVE"
      },
      "isHidden": false,
      "preselectDefaultProduct": false,
      "notPartOfOriginalCombo": false
    }
  ],
  "productCode": 6509785,
  "productImages": [
    "https://cfg-japan-east-prod-1.azureedge.net/xy/1080/1080/?path=op29876_en-7.png&tId=LJP705"
  ],
  "inOutage": false,
  "localisedComboName": {
    "cartName": "チキンマックナゲットハッピーセット\r\n（マックフライポテトＳ＋ドリンクＳ＋本またはおもちゃ）\r\n＋募金￥１０",
    "menuName": "チキンマックナゲットハッピーセット\r\n（マックフライポテトＳ＋ドリンクＳ＋本またはおもちゃ）\r\n＋募金￥１０",
    "variantName": "チキンマックナゲットハッピーセット\r\n（マックフライポテトＳ＋ドリンクＳ＋本またはおもちゃ）\r\n＋募金￥１０"
  },
  "offerId": "29876"
}
```



## 获取产品里面的配料（非产品）

根据上面数据里面的 productCode(6509785) 以及 choiceProductCode 获取其他非产品数据（可选项数据）

```json
[
  {
    "additionalChoices": [],
    "categoryId": 5,
    "productCode": 3472,
    "displayOrder": 6280,
    "productImages": [
      "https://mopfd.plexure.io/product-images/product_mobile/3472"
    ],
    "inOutage": false,
    "recommended": false,
    "localisedProductName": {
      "cartName": "Sokenbicha (S)",
      "menuName": "Sokenbicha",
      "variantName": "S size"
    },
    "dayPart": "BREAKFAST_DAY_MENU",
    "productOptions": [
      {
        "localisedProductName": {
          "cartName": "Sokenbicha (S)",
          "menuName": "Sokenbicha",
          "variantName": "S size"
        },
        "dayPart": "BREAKFAST_DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3472
      }
    ],
    "priceList": [
      {
        "price": 103,
        "priceCode": "EATIN"
      },
      {
        "price": 103,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 103,
        "priceCode": "OTHER"
      }
    ],
    "priceDifference": [
      {
        "price": 0,
        "priceCode": "EATIN"
      },
      {
        "price": 0,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 0,
        "priceCode": "OTHER"
      }
    ],
    "productClass": "PRODUCT",
    "status": "ACTIVE"
  },
  {
    "additionalChoices": [],
    "categoryId": 5,
    "productCode": 3917,
    "displayOrder": 6440,
    "productImages": [
      "https://mopfd.plexure.io/product-images/product_mobile/3917"
    ],
    "inOutage": false,
    "recommended": false,
    "localisedProductName": {
      "cartName": "Yasaiseikatsu100(S)",
      "menuName": "Yasaiseikatsu100",
      "variantName": "S size"
    },
    "dayPart": "BREAKFAST_DAY_MENU",
    "productOptions": [
      {
        "localisedProductName": {
          "cartName": "Yasaiseikatsu100(S)",
          "menuName": "Yasaiseikatsu100",
          "variantName": "S size"
        },
        "dayPart": "BREAKFAST_DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3917
      }
    ],
    "priceList": [
      {
        "price": 103,
        "priceCode": "EATIN"
      },
      {
        "price": 103,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 103,
        "priceCode": "OTHER"
      }
    ],
    "priceDifference": [
      {
        "price": 0,
        "priceCode": "EATIN"
      },
      {
        "price": 0,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 0,
        "priceCode": "OTHER"
      }
    ],
    "productClass": "PRODUCT",
    "status": "ACTIVE"
  },
  {
    "additionalChoices": [],
    "categoryId": 5,
    "productCode": 3310,
    "displayOrder": 6470,
    "productImages": [
      "https://mopfd.plexure.io/product-images/product_mobile/3310"
    ],
    "inOutage": false,
    "recommended": false,
    "localisedProductName": {
      "cartName": "Minute Maid Orange (S)",
      "menuName": "Minute Maid Orange",
      "variantName": "S size"
    },
    "dayPart": "BREAKFAST_DAY_MENU",
    "productOptions": [
      {
        "localisedProductName": {
          "cartName": "Minute Maid Orange (S)",
          "menuName": "Minute Maid Orange",
          "variantName": "S size"
        },
        "dayPart": "BREAKFAST_DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3310
      }
    ],
    "priceList": [
      {
        "price": 103,
        "priceCode": "EATIN"
      },
      {
        "price": 103,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 103,
        "priceCode": "OTHER"
      }
    ],
    "priceDifference": [
      {
        "price": 0,
        "priceCode": "EATIN"
      },
      {
        "price": 0,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 0,
        "priceCode": "OTHER"
      }
    ],
    "productClass": "PRODUCT",
    "status": "ACTIVE"
  },
  {
    "additionalChoices": [],
    "categoryId": 5,
    "productCode": 3922,
    "displayOrder": 6500,
    "productImages": [
      "https://mopfd.plexure.io/product-images/product_mobile/3922"
    ],
    "inOutage": false,
    "recommended": false,
    "localisedProductName": {
      "cartName": "Minute Maid Apple100",
      "menuName": "Minute Maid Apple100",
      "variantName": "Minute Maid Apple100"
    },
    "dayPart": "BREAKFAST_DAY_MENU",
    "productOptions": [],
    "priceList": [
      {
        "price": 103,
        "priceCode": "EATIN"
      },
      {
        "price": 103,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 103,
        "priceCode": "OTHER"
      }
    ],
    "priceDifference": [
      {
        "price": 0,
        "priceCode": "EATIN"
      },
      {
        "price": 0,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 0,
        "priceCode": "OTHER"
      }
    ],
    "productClass": "PRODUCT",
    "status": "ACTIVE"
  },
  {
    "additionalChoices": [],
    "categoryId": 5,
    "productCode": 3480,
    "displayOrder": 6510,
    "productImages": [
      "https://mopfd.plexure.io/product-images/product_mobile/3480"
    ],
    "inOutage": false,
    "recommended": false,
    "localisedProductName": {
      "cartName": "Milk",
      "menuName": "Milk",
      "variantName": "Milk"
    },
    "dayPart": "BREAKFAST_DAY_MENU",
    "productOptions": [],
    "priceList": [
      {
        "price": 103,
        "priceCode": "EATIN"
      },
      {
        "price": 103,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 103,
        "priceCode": "OTHER"
      }
    ],
    "priceDifference": [
      {
        "price": 0,
        "priceCode": "EATIN"
      },
      {
        "price": 0,
        "priceCode": "TAKEOUT"
      },
      {
        "price": 0,
        "priceCode": "OTHER"
      }
    ],
    "productClass": "PRODUCT",
    "status": "ACTIVE"
  },
  {
    "categoryId": 0,
    "productCode": 0,
    "displayOrder": 0,
    "productImages": [
      "https://mopdevstores.blob.core.windows.net/category-images/category_mobile/7"
    ],
    "inOutage": false,
    "recommended": false,
    "localisedProductName": {
      "cartName": "Other drinks",
      "menuName": "Other drinks",
      "variantName": "Other drinks"
    },
    "dayPart": "DAY_MENU",
    "productOptions": [
      {
        "localisedProductName": {
          "cartName": "Coca Cola (S)",
          "menuName": "Coca Cola",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3110
      },
      {
        "localisedProductName": {
          "cartName": "Fanta Grape (S)",
          "menuName": "Fanta Grape",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3400
      },
      {
        "localisedProductName": {
          "cartName": "Qoo White Grape (S)",
          "menuName": "Qoo White Grape",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3603
      },
      {
        "localisedProductName": {
          "cartName": "McShake® Chocolate (S)",
          "menuName": "McShake® Chocolate (S)",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3020
      },
      {
        "localisedProductName": {
          "cartName": "McShake® Strawberry (S)",
          "menuName": "McShake® Strawberry (S)",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3010
      },
      {
        "localisedProductName": {
          "cartName": "McShake® Vanilla (S)",
          "menuName": "McShake® Vanilla (S)",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3030
      },
      {
        "localisedProductName": {
          "cartName": "Coca Cola Zero (S)",
          "menuName": "Coca Cola Zero",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3331
      },
      {
        "localisedProductName": {
          "cartName": "Sprite (S)",
          "menuName": "Sprite",
          "variantName": "S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3160
      },
      {
        "localisedProductName": {
          "cartName": "Fanta Melon (S)",
          "menuName": "Fanta Melon",
          "variantName": " S size"
        },
        "dayPart": "DAY_MENU",
        "optionType": "AlternateSize",
        "productCode": 3415
      }
    ],
    "priceList": [],
    "priceDifference": [],
    "productClass": "PRODUCT",
    "status": "ACTIVE"
  }
]
```



# https://adv-japan-e-staging.vmobapps.com 

## /v3/advertisements?channel=MAN&offset=0&limit=100&ignoreDailyTimeFilter=false

### 获取首页广告

```json
[{
	"channelCode": "MAN",
	"clickThroughUrl": "mcdonaldsjp://webview?url=https://tick.tmeiju.com/products_test.html",
	"closestVenue": null,
	"contentTagReferenceCodes": [],
	"contentUrl": "",
	"dailyEndTime": null,
	"dailyStartTime": null,
	"dateCreated": "2017-01-27T15:02:29Z",
	"dateModified": "2020-12-08T03:38:51Z",
	"daysOfWeek": [0, 1, 2, 3, 4, 5, 6],
	"description": "ついに本日から決勝戦! ダブチ vs. テリヤキ １位になるのはどっち...!?",
	"distanceToClosestVenue": null,
	"endDate": "2022-01-29T04:00:00Z",
	"eventStartDateTime": null,
	"extendedData": null,
	"id": 1679,
	"image": "asi1679_high_en-141.png",
	"imageDescription": "",
	"isActive": true,
	"isAvailableAllStores": true,
	"merchantId": 563,
	"placementCode": "NR",
	"startDate": "2017-02-03T04:00:00Z",
	"title": "2017/01/16",
	"venues": null,
	"weight": 0
}, {
	"channelCode": "MAN",
	"clickThroughUrl": "mcdonaldsjp://webapp?webAppUrl=https://tick.tmeiju.com/products_test.html",
	"closestVenue": null,
	"contentTagReferenceCodes": [],
	"contentUrl": "",
	"dailyEndTime": null,
	"dailyStartTime": null,
	"dateCreated": "2020-12-09T06:34:53Z",
	"dateModified": "2020-12-09T06:40:36Z",
	"daysOfWeek": [0, 1, 2, 3, 4, 5, 6],
	"description": "ついに本日から決勝戦! ダブチ vs. テリヤキ １位になるのはどっち...!?",
	"distanceToClosestVenue": null,
	"endDate": "2022-01-29T04:00:00Z",
	"eventStartDateTime": null,
	"extendedData": null,
	"id": 1886,
	"image": "asi1886_high_en-14.png",
	"imageDescription": "",
	"isActive": true,
	"isAvailableAllStores": true,
	"merchantId": 563,
	"placementCode": "NR",
	"startDate": "2017-02-03T04:00:00Z",
	"title": "Test Webapp",
	"venues": null,
	"weight": 0
}, {
	"channelCode": "MAN",
	"clickThroughUrl": "mcdonaldsjp://cpn-vlstmp2102?cid=ca03f28b-961d-44ab-bc94-362a086b905d&lcpf=ValueLunch_2102&target_tag=202102_VLMOP_Target",
	"closestVenue": null,
	"contentTagReferenceCodes": [],
	"contentUrl": "",
	"dailyEndTime": null,
	"dailyStartTime": null,
	"dateCreated": "2021-01-07T02:33:12Z",
	"dateModified": "2021-01-08T04:18:35Z",
	"daysOfWeek": [0, 1, 2, 3, 4, 5, 6],
	"description": "Test Value Stamp Card",
	"distanceToClosestVenue": null,
	"endDate": "2022-02-28T04:00:00Z",
	"eventStartDateTime": null,
	"extendedData": null,
	"id": 1888,
	"image": "asi1888_high_en-34.png",
	"imageDescription": "",
	"isActive": true,
	"isAvailableAllStores": true,
	"merchantId": 563,
	"placementCode": "NBS",
	"startDate": "2021-01-05T04:00:00Z",
	"title": "Value Stamp Card",
	"venues": null,
	"weight": 0
}, {
	"channelCode": "MAN",
	"clickThroughUrl": "mcdonaldsjp://cpn-vlstmp2102?cid=ca03f28b-961d-44ab-bc94-362a086b905d&lcpf=ValueLunch_2102&end=2021-03-12T13:59:00+09:00&target_tag=202102_VLMOP_Target",
	"closestVenue": null,
	"contentTagReferenceCodes": [],
	"contentUrl": "",
	"dailyEndTime": null,
	"dailyStartTime": null,
	"dateCreated": "2021-01-07T02:35:12Z",
	"dateModified": "2021-01-08T04:18:58Z",
	"daysOfWeek": [0, 1, 2, 3, 4, 5, 6],
	"description": "",
	"distanceToClosestVenue": null,
	"endDate": "2022-02-28T04:00:00Z",
	"eventStartDateTime": null,
	"extendedData": null,
	"id": 1889,
	"image": "asi1889_high_en-38.png",
	"imageDescription": "",
	"isActive": true,
	"isAvailableAllStores": true,
	"merchantId": 563,
	"placementCode": "NBS",
	"startDate": "2021-01-05T04:00:00Z",
	"title": "Value Stamp Card End",
	"venues": null,
	"weight": 0
}]

mcdonaldsjp://cpn-vlstmp2102?cid={identifier}&lcpf=ValueLunch_2102&end=2021-03-12T13:59:00+09:00&target_tag=202102_VLMOP_Target
mcdonaldsjp://cpn-vlstmp2102?cid={identifier}&lcpf=ValueLunch_2102&target_tag=202102_VLMOP_Target&end=2021-01-06T13:59:00+09:00
```



# https://cfg-japan-e-staging.vmobapps.com

## /v3/configurations

### 获取app 配置，用于判断是否要进行维护页面显示

```json
{
	"activitiesBatchSize": 30,
	"activitiesExpireAgeInDays": 60,
	"activitiesExpireSize": 1000,
	"activitiesSendIntervalMinutes": 30,
	"activitiesSendMinTimerInSeconds": 30,
	"activityApiUrl": "https://act-Japan-E-Staging.vmobapps.com/v3",
	"advertisementApiUrl": "https://adv-Japan-E-Staging.vmobapps.com/v3",
	"advertisementImagePrefix": "https://cfg-japan-e-staging.azureedge.net",
	"assetsDownloadPrefix": "https://cfg-japan-e-staging.azureedge.net/downloads",
	"authorizationApiUrl": "https://authorization-vmob-stage-jpe.vmobapps.com/",
	"activityV2ApiUrl": null,
	"beaconRegions": "61687109-905f-4436-91f8-e602f514c96d,1acbad6e-e1a5-4838-a62a-22d35d00c35b",
	"blackoutEndsUTCMinutesFromMidnight": 1380,
	"blackoutStartsUTCMinutesFromMidnight": 900,
	"categories": [{
		"highResolutionImage": "categories/11/category_11_high_notselected_alldevices.png?version=100",
		"highResolutionSelectedImage": "categories/11/category_11_high_selected_alldevices.png?version=100",
		"id": 11,
		"lowResolutionImage": "categories/11/category_11_low_notselected_alldevices.png?version=100",
		"lowResolutionSelectedImage": "categories/11/category_11_low_selected_alldevices.png?version=100",
		"name": "Miseru Coupon",
		"sortOrder": 0
	}, {
		"highResolutionImage": "categories/1/category_1_high_notselected_alldevices.png?version=100",
		"highResolutionSelectedImage": "categories/1/category_1_high_selected_alldevices.png?version=100",
		"id": 1,
		"lowResolutionImage": "categories/1/category_1_low_notselected_alldevices.png?version=100",
		"lowResolutionSelectedImage": "categories/1/category_1_low_selected_alldevices.png?version=100",
		"name": "Default",
		"sortOrder": 1
	}, {
		"highResolutionImage": "categories/12/category_12_high_notselected_alldevices.png?version=100",
		"highResolutionSelectedImage": "categories/12/category_12_high_selected_alldevices.png?version=100",
		"id": 12,
		"lowResolutionImage": "categories/12/category_12_low_notselected_alldevices.png?version=100",
		"lowResolutionSelectedImage": "categories/12/category_12_low_selected_alldevices.png?version=100",
		"name": "Kazasu Coupon",
		"sortOrder": 5
	}],
	"categoryImagePrefix": "https://cfg-japan-e-staging.azureedge.net",
	"configurationApiUrl": "https://cfg-Japan-E-Staging.vmobapps.com/v3",
	"consumerApiUrl": "https://con-Japan-E-Staging.vmobapps.com/v3",
	"extendedParameters": "{\"LowAndroidVersion\":\"40019\",\"LowIOSVersion\":\"2.18.12\",\"InitialRedemptionCounterMinutes\":\"5\",\"verificationOverideToken\":\"a84b380f-526a-49fe-901b-15e3250951be\",\"maintenanceModeUrl\":\"https://tvk048maintenance.blob.core.windows.net/vmob/maintcfg.json\"}",
	"geofenceBlackoutWindowOffset": 60,
	"geoTileSizeInDegrees": 0.1,
	"gpsBackgroundMinIntervalInSeconds": 301,
	"gpsPollIntervalInSeconds": 300,
	"locationAccuracy": 100,
	"locationApiUrl": "https://loc-Japan-E-Staging.vmobapps.com/v3",
	"locationBackgroundSourceDefault": "gps",
	"locationDistanceFilter": 700,
	"locationEnableBackgroundGps": true,
	"offerApiUrl": "https://off-Japan-E-Staging.vmobapps.com/v3",
	"offerExtensionsApiUrl": "",
	"offerImagePrefix": "https://cfg-japan-e-staging.azureedge.net",
	"redeemedOfferImagePrefix": "https://cfg-japan-e-staging.azureedge.net",
	"verificationAddressConsumerEmailChange": "ConsumerEmailUpdateVerification@tvk048-inbound.vmobapps.com",
	"verificationAddressLogin": "LoginVerification@tvk048-inbound.vmobapps.com",
	"verificationAddressRegistration": "RegistrationVerification@tvk048-inbound.vmobapps.com",
	"verificationDeepLinkURLSchemeName": "vmobsdkmcdjapan",
	"tenantId": "TVK048",
	"merchantMfaSettings": [{
		"merchantId": 563,
		"merchantName": "McDonalds Japan",
		"status": "Disabled",
		"supportedChannels": ["Email"]
	}, {
		"merchantId": 565,
		"merchantName": "test",
		"status": "Disabled",
		"supportedChannels": ["Email"]
	}, {
		"merchantId": 567,
		"merchantName": "Test Merchant 2016-10-17",
		"status": "Disabled",
		"supportedChannels": ["Email"]
	}, {
		"merchantId": 568,
		"merchantName": "Runscope Merchant (Do Not Give to Client View)",
		"status": "Disabled",
		"supportedChannels": ["Email"]
	}, {
		"merchantId": 571,
		"merchantName": "Plexure Japan DEMO",
		"status": "Disabled",
		"supportedChannels": ["Email"]
	}],
	"applicationSettings": {
		"applicationVersions": {
			"android": {
				"minimumVersion": null,
				"appStoreUrl": null
			},
			"iOS": {
				"minimumVersion": null,
				"appStoreUrl": null
			}
		}
	},
	"mobileAppFeatureFlags": null
}
```



# https://con

## /customer

### 获取更新用户信息(get、put)

```json
{
	"firstName": "",
	"lastName": "",
	"fullName": " ",
	"gender": "",
	"dateOfBirth": "",
	"homeCity": null,
	"emailAddress": "john@theplant.jp",
	"userName": "john@theplant.jp",
	"extendedData": "{\"news_coupon\":\"0\",\"isConnectedFacebook\":\"0\",\"isConnectedTwitter\":\"0\",\"isRegistered\":\"1\",\"newNGLP2appUser\":true,\"num_children\":0,\"new_products\":\"0\",\"pushNewsCampain\":\"0\",\"special_gift\":\"0\"}",
	"pushMessageOptOut": false,
	"postcode": null,
	"mobileNumber": null,
	"currentMerchant": null,
	"isMfaEnabled": false,
	"mfaChannel": null
}
```

```
// put
{
	"emailAddress": "john@theplant.jp",
	"extendedData": "{\"news_coupon\":\"0\",\"isConnectedFacebook\":\"0\",\"isConnectedTwitter\":\"0\",\"isRegistered\":\"1\",\"newNGLP2appUser\":true,\"num_children\":0,\"new_products\":\"0\",\"pushNewsCampain\":\"0\",\"special_gift\":\"0\"}",
	"firstName": "",
	"lastName": "",
	"userName": "john@theplant.jp"
}
```



## /consumers/me/tagvalues:

### 获取更新用户标签（get、put）

```json
{
	"tagValueReferenceCodes": ["mop_user", "profile_num_children_0", "bluetooth_disabled", "LastOpenedApp_0_1_Days", "profile_news_coupon_no", "SampleGroup7", "LastRedeemed_0_6_Days", "device-os-android", "profile_new_products_no", "profile_special_gift_no", "android_miseru", "Early_Engagement_201604", "Region_X", "location_service_disabled", "push_messages_disabled"]
}
```

```
// put
{
	"tagValueRemoveReferenceCodes": [],
	"tagValueAddReferenceCodes": ["NGLP2"]
}
```



# https://off-japan-e-staging.vmobapps.com

## /v3/loyaltycards?tagExpression=

### 获取活动的 loyaltycards

```json
[{
	"categoryId": 11,
	"categoryName": "Miseru Coupon",
	"loyaltyCardId": 237,
	"dailyStartTime": null,
	"dailyEndTime": null,
	"maxInstances": 1000,
	"startDate": "2020-08-31T15:00:00Z",
	"endDate": "2022-09-29T15:00:00Z",
	"extendedData": null,
	"title": "MOP POS Card Sept 11",
	"subtitle": "MOP POS Card Sept 11",
	"description": "MOP POS Card Sept 11",
	"instructions": "",
	"termsAndConditions": "",
	"pointsRequired": 105,
	"pointsText": "",
	"weighting": 0,
	"cardImage": "loyaltycard/237/loyaltycard_237_high_en-12.png",
	"cardImageDescription": "",
	"assetsPath": "loyaltycard/237/assets/",
	"instancesAvailable": 0,
	"maxPointsPerDay": 1000,
	"maxPointsRequestsPerDay": null,
	"initialPoints": 10,
	"isActive": true,
	"reasonCodes": [],
	"instances": [],
	"offers": [{
		"burntCount": 0,
		"closestVenue": null,
		"product": null,
		"products": null,
		"distanceToClosestVenue": null,
		"giftBatchId": null,
		"giftedBy": null,
		"giftedByConsumerId": null,
		"giftedCopy": null,
		"giftedOnDate": null,
		"giftId": null,
		"isActive": true,
		"isAGift": false,
		"isMerchantFavourite": false,
		"lastBurntAt": null,
		"lastRedeemedAt": null,
		"offerInstanceUniqueId": "",
		"redemptionCount": 0,
		"totalRemainingRedemptionCount": null,
		"respawnLimit": null,
		"codeExpiryInMinutes": 10,
		"pointValue": null,
		"isReserved": null,
		"offerReservation": null,
		"categoryId": 11,
		"categoryName": "Miseru Coupon",
		"codeType": 4,
		"contentTagReferenceCodes": [],
		"contentUrl": "",
		"dailyEndTime": null,
		"dailyStartTime": null,
		"daysOfWeek": [],
		"description": "POS Loyalty Offer",
		"endDate": "2021-10-01T18:55:00Z",
		"extendedData": "This is a template",
		"id": 3625,
		"image": null,
		"imageAlt": null,
		"imageAltDescription": "Alternative Image description",
		"imageDescription": "Main Image description",
		"isAvailableAllStores": true,
		"isGiftable": false,
		"isPremiumPlacement": false,
		"isRespawningOffer": false,
		"isReward": true,
		"isSticky": false,
		"lastUpdatedAt": "2021-01-07T02:59:21Z",
		"merchantId": 563,
		"paymentAmount": null,
		"paymentTaxRate": null,
		"paymentType": 0,
		"respawnsInDays": 0,
		"respawnStartTime": null,
		"sortOrder": 0,
		"startDate": "2020-08-31T17:00:00Z",
		"termsAndConditions": null,
		"title": "POS  loyalty short code offer",
		"venueExternalIds": null,
		"venueIds": null,
		"weighting": 0
	}],
	"contentTagReferenceCodes": [],
	"loyaltyCardType": "StampCard",
	"pointsExpiryDays": null,
	"expiryScheduleDetails": "EndOfMonth"
}, {
	"categoryId": 11,
	"categoryName": "Miseru Coupon",
	"loyaltyCardId": 248,
	"dailyStartTime": null,
	"dailyEndTime": null,
	"maxInstances": 1000,
	"startDate": "2020-12-31T15:00:00Z",
	"endDate": "2021-03-30T15:00:00Z",
	"extendedData": null,
	"title": "ValueLunch_2102",
	"subtitle": "ValueLunch_2102",
	"description": "ValueLunch_2102\r\n",
	"instructions": "",
	"termsAndConditions": "",
	"pointsRequired": 4,
	"pointsText": "",
	"weighting": 0,
	"cardImage": "loyaltycard/248/loyaltycard_248_high_en-6.png",
	"cardImageDescription": "",
	"assetsPath": "loyaltycard/248/assets/",
	"instancesAvailable": 0,
	"maxPointsPerDay": null,
	"maxPointsRequestsPerDay": null,
	"initialPoints": 0,
	"isActive": true,
	"reasonCodes": [],
	"instances": [{
		"loyaltyCardId": 248,
		"instanceId": "17dbb5b7-8f59-496b-a9bf-fe0bf40e151f",
		"isActive": false,
		"pointsBalance": 4,
		"redeemedOfferId": 3667
	}, {
		"loyaltyCardId": 248,
		"instanceId": "282d4c02-8c81-40dd-add2-247cae7ec4ef",
		"isActive": false,
		"pointsBalance": 4,
		"redeemedOfferId": 3667
	}, {
		"loyaltyCardId": 248,
		"instanceId": "f9dc50e2-4527-4704-973f-ff5236c17262",
		"isActive": true,
		"pointsBalance": 1,
		"redeemedOfferId": null
	}],
	"offers": [{
		"burntCount": 0,
		"closestVenue": null,
		"product": null,
		"products": null,
		"distanceToClosestVenue": null,
		"giftBatchId": null,
		"giftedBy": null,
		"giftedByConsumerId": null,
		"giftedCopy": null,
		"giftedOnDate": null,
		"giftId": null,
		"isActive": true,
		"isAGift": false,
		"isMerchantFavourite": false,
		"lastBurntAt": null,
		"lastRedeemedAt": null,
		"offerInstanceUniqueId": "4c1c6c0a-87fd-4aeb-a721-0cb5ba9f52cd",
		"redemptionCount": 0,
		"totalRemainingRedemptionCount": null,
		"respawnLimit": null,
		"codeExpiryInMinutes": null,
		"pointValue": null,
		"isReserved": null,
		"offerReservation": null,
		"categoryId": 11,
		"categoryName": "Miseru Coupon",
		"codeType": 1,
		"contentTagReferenceCodes": [],
		"contentUrl": "",
		"dailyEndTime": null,
		"dailyStartTime": null,
		"daysOfWeek": [],
		"description": "※1回の提示につき1個までご利用いただけます。\r\n※画像はイメージです。",
		"endDate": "2021-03-31T04:00:00Z",
		"extendedData": "{\r\n\t\"typeText\":\"￥100\",\r\n\t\"offerType\":\"1回限り\",\r\n\t\"offerTypeColor\":\"Gold\",\r\n\t\"productCode\":32009715\r\n}",
		"id": 3667,
		"image": "op3667_en-4.png",
		"imageAlt": null,
		"imageAltDescription": "",
		"imageDescription": "",
		"isAvailableAllStores": true,
		"isGiftable": false,
		"isPremiumPlacement": false,
		"isRespawningOffer": false,
		"isReward": true,
		"isSticky": false,
		"lastUpdatedAt": "2021-01-07T02:59:21Z",
		"merchantId": 563,
		"paymentAmount": null,
		"paymentTaxRate": null,
		"paymentType": 0,
		"respawnsInDays": 0,
		"respawnStartTime": null,
		"sortOrder": 0,
		"startDate": "2021-01-01T04:00:00Z",
		"termsAndConditions": null,
		"title": "ナゲット100円",
		"venueExternalIds": null,
		"venueIds": null,
		"weighting": 0
	}],
	"contentTagReferenceCodes": [],
	"loyaltyCardType": "StampCard",
	"pointsExpiryDays": null,
	"expiryScheduleDetails": "EndOfMonth"
}]
```

