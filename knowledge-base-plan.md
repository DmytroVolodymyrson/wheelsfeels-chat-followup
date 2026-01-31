# База знаний WheelsFeels - Краткое описание для клиента

---

## Что сейчас

Ссылки на продукты зашиты прямо в промпт AI:
- 27 типов авто с годами и конфигурациями
- 50+ продуктов с URL
- Работает, но сложно поддерживать при росте

---

## Проблема

Добавить мануалы так же (в промпт) можно, но:
- Промпт станет очень большим
- Каждое изменение = редактирование workflow
- Нет поиска
- Сложно масштабировать

---

## Решение: База данных в Supabase

**Почему Supabase:**
- Уже используется для логирования чатов
- Легко добавлять/редактировать через веб-интерфейс
- Можно расширять (FAQ, видео, спецификации)
- Автоматическая синхронизация с сайтом (опционально)

**Как будет работать:**
1. Все продукты + мануалы в одной таблице
2. AI запрашивает базу → получает продукт + мануал
3. Генерирует email с обеими ссылками

---

## Вопросы к клиенту

### Обязательные:

1. **Сколько всего продуктов/авто поддерживаете?**
   - Нужно для выбора подхода

2. **Где сейчас хранятся мануалы?**
   - Прямые URL на сайте?
   - PDF где-то отдельно?
   - Нужно куда-то загрузить?

3. **Один мануал на продукт или несколько?**
   - Только инструкция по установке?
   - Еще гарантия, технические характеристики?

### Для планирования:

4. **Кто будет обновлять базу знаний?**
   - Технический человек (может работать с Supabase)?
   - Нужен простой интерфейс/форма?
   - Импорт из Google Sheets?

5. **Нужна ли автоматическая синхронизация с сайтом?**
   - Или достаточно ручных обновлений?

6. **Что еще кроме мануалов может понадобиться?**
   - FAQ, видео, спецификации?
   - Это влияет на структуру базы

---

## Что получим

✅ Единая база знаний для всех продуктов
✅ Легкое обновление без изменения automation
✅ Мануалы вместе с продуктами в emails
✅ Готовность к расширению (chatbot на сайте, FAQ)
✅ Возможность ручного добавления информации

---

# Technical Details (for implementation)

---

## Recommendation Summary

Based on current needs and mentioned expansion plans:

| Criteria | Option A (Prompt) | Option B (Supabase DB) | Option C (RAG) |
|----------|-------------------|------------------------|----------------|
| Implementation effort | Low | Medium | High |
| Scalability | Limited (~100 items) | High (1000s items) | Very High |
| Maintenance ease | Hard (code changes) | Easy (database) | Medium |
| Future expansion | Limited | Excellent | Excellent |
| For website chatbot | No | Yes | Yes |
| Cost | Lowest | Low | Medium |

**Recommendation:** Option B (Supabase Product Database)
- Matches client's existing Supabase experience
- Provides foundation for future expansion
- Easy to maintain and extend
- Can be enhanced to Option C later if needed
- Even for small catalogs, easier to maintain than prompt-based approach

---

## Simplified Questions for Client

### Critical (need answers before starting):

1. **How many products/vehicles do you support total?**
   - This helps determine if simple prompt extension would work or if database is necessary

2. **Where are manuals currently stored?**
   - Direct URLs we can add?
   - Need to upload somewhere first?
   - Format (PDF, webpage, video)?

3. **One manual per product, or different manuals for same product?**
   - Installation guide, user manual, warranty info, etc.?

### Important (shapes implementation):

4. **Who will maintain the knowledge base?**
   - Technical person comfortable with Supabase dashboard?
   - Non-technical - need n8n form interface?
   - Spreadsheet import from Google Sheets?

5. **Should website data auto-sync to knowledge base?**
   - Or manual updates are fine?

6. **What else beyond manuals might be added later?**
   - FAQs, specs, videos, installation guides?
   - Helps us design schema correctly upfront

---

## Implementation Plan (Once Questions Answered)

### Phase 1: Database Setup
1. Create Supabase `products` table with schema:
   - vehicle info (make, model, year range, generation, config)
   - product_name, product_url, manual_url
   - optional: description, category, in_stock

2. Migrate existing 27 vehicles + 50 products from prompt to database

3. Add manual URLs to each product record

### Phase 2: Workflow Integration
4. Create n8n sub-workflow: "Lookup Product & Manual"
   - Input: vehicle make, model, year, config
   - Query Supabase for matching products
   - Return: product URL, manual URL, description

5. Update main workflow's Product Lookup tool to call sub-workflow instead of using hardcoded prompt

6. Update email generation prompt to include manual links when relevant

### Phase 3: Maintenance Interface
7. Option A: Use Supabase dashboard directly
8. Option B: Create n8n form for adding/editing products
9. Option C: Google Sheets import workflow

### Phase 4: Optional Enhancements
- Website webhook to auto-update products
- Notification when product data changes
- Admin panel in n8n

---

## Verification Plan
1. Test database queries with various vehicle combinations
2. Verify email generation includes correct product AND manual links
3. Test edge cases (vehicle not in database, no manual available)
4. Compare output quality before/after migration
