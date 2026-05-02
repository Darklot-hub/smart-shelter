# Сущности и связи

## 1. Animal (животное) — расширенная
| Поле | Тип | Описание |
|------|-----|----------|
| id | uuid | PK |
| name | string | кличка |
| type | enum('cat','dog') | |
| age | number | месяцев |
| photoUrl | string | |
| **status** | **enum** | IN_SHELTER, QUARANTINE, TREATMENT, READY_FOR_ADOPTION, ADOPTED |
| **kidFriendly** | boolean | дружелюбен к детям |
| **petFriendly** | boolean | дружелюбен к другим животным |
| **spaceNeed** | enum('small','medium','large') | потребность в пространстве |
| **energyLevel** | enum('low','medium','high') | активность |
| **experienceRequired** | enum('none','beginner','experienced') | требуемый опыт |
| createdAt | timestamp | |
| updatedAt | timestamp | |

## 2. User (пользователь) — для аутентификации и ролей
| Поле | Тип |
|------|-----|
| id | uuid |
| email | string |
| passwordHash | string |
| role | enum('admin','staff','volunteer','adopter') |
| name | string |

## 3. Questionnaire (анкета усыновителя)
| Поле | Тип |
|------|-----|
| id | uuid |
| userId | uuid (FK → User, может быть null для гостя) |
| hasChildren | boolean |
| hasOtherPets | boolean |
| otherPetsTypes | string |
| housingType | enum('apartment','house') |
| apartmentSize | enum('small','medium','large') |
| experienceLevel | enum('none','beginner','experienced') |
| createdAt | timestamp |

## 4. AdoptionRequest (заявка)
| Поле | Тип |
|------|-----|
| id | uuid |
| animalId | uuid (FK → Animal) |
| userId | uuid (FK → User, опционально для гостя) |
| questionnaireId | uuid (FK → Questionnaire) |
| status | enum('pending','approved','rejected') |
| createdAt | timestamp |

## 5. Task (волонтёрская задача)
| Поле | Тип |
|------|-----|
| id | uuid |
| animalId | uuid (FK → Animal, nullable) |
| assignedTo | uuid (FK → User, где role='volunteer') |
| description | text |
| scheduledTime | timestamp |
| status | enum('open','taken','completed') |
| type | enum('walk','clean','vet_visit','other') |

## 6. Notification (уведомление)
| Поле | Тип |
|------|-----|
| id | uuid |
| userId | uuid (FK → User) |
| type | enum('adoption_approved','task_assigned','task_reminder','vaccination_expiring') |
| title | string |
| message | text |
| isRead | boolean |
| createdAt | timestamp |

## 7. Vaccination (прививка — опциональная сущность)
| Поле | Тип |
|------|-----|
| id | uuid |
| animalId | uuid (FK → Animal) |
| vaccineName | string |
| dateGiven | date |
| validUntil | date |
| notes | text |

## Бизнес-логика (реализуется в коде)
- **`calculateCompatibility(questionnaire, animal)`** → возвращает число от 0 до 100.  
  Критерии: kidFriendly (25%), petFriendly (20%), spaceNeed (25%), energyLevel (15%), experienceRequired (15%).
- **Правила смены статуса:**  
  `READY_FOR_ADOPTION` → `ADOPTED` только при наличии одобренной заявки.  
  Нельзя сменить `ADOPTED` на другой статус.
- **Авто-уведомления:**  
  – При одобрении заявки: email + in-app уведомление усыновителю.  
  – При создании задачи: уведомление волонтёру.