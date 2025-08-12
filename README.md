# /src/utils/libs

## getPosts.ts || getAllPosts.ts
```ts
import { getCollection } from "astro:content"

// Вывод постов по новым датам и по статусу draft
export const getAllPosts = async content => {
	const posts = (await getCollection(content))
		.filter(({ data }) => (import.meta.env.PROD ? data.draft !== true : true))
		// "2024-02-21T15:30:00Z"
		//	datePublished: z.union([z.string().datetime(), z.date()]),
		.sort((a, b) => +new Date(b.data.pubDate) - +new Date(a.data.pubDate))
	return posts
}

```

## getTaxonomy.ts
```ts
import { getAllPosts } from "./getPosts"
// Функция для получения всех таксономий
const getTaxonomy = async (collection, name) => {
	const singlePages = await getAllPosts(collection)
	let taxonomies = []

	// Сбор всех таксономий из постов
	singlePages.forEach(page => {
		const tagArray = page.data[name] // Получаем массив категорий из каждого поста
		if (Array.isArray(tagArray)) {
			// Проверяем, является ли это массивом
			tagArray.forEach(tag => {
				// Добавляем слаг в массив
				taxonomies.push(tag) // Используем slugify для нормализации
			})
		}
	})

	// Удаляем дубликаты с помощью Set
	const uniqueTaxonomies = [...new Set(taxonomies)]
	return uniqueTaxonomies
}

export default getTaxonomy

```

## taxonomyFilter
```ts
import { slugify } from "./utils"

const taxonomyFilter = (posts: any[], name: string, key: any) =>
	posts.filter(
		post =>
			Array.isArray(post.data[name]) &&
			post.data[name].map((name: string) => slugify(name)).includes(key),
	)

export default taxonomyFilter
```

## slugify и slugifyReverse (для тегов, [tag].astro и прочего)
```ts
export function slugify(input) {
	if (!input) return ""

	const ruToEn = {
		а: "a",
		б: "b",
		в: "v",
		г: "g",
		д: "d",
		е: "e",
		ё: "yo",
		ж: "zh",
		з: "z",
		и: "i",
		й: "y",
		к: "k",
		л: "l",
		м: "m",
		н: "n",
		о: "o",
		п: "p",
		р: "r",
		с: "s",
		т: "t",
		у: "u",
		ф: "f",
		х: "kh",
		ц: "ts",
		ч: "ch",
		ш: "sh",
		щ: "shch",
		ы: "y`", // y с бэктиком
		э: "e`",
		ю: "yu",
		я: "ya",
		ъ: "`",
		ь: "'",
	}

	let slug = input
		.toLowerCase()
		.trim()
		.replace(/[а-яё]/gi, match => ruToEn[match] || match)

	slug = slug.normalize("NFD").replace(/\p{M}/gu, "")

	slug = slug.replace(/[^a-z0-9\s-`']/g, " ").trim()

	slug = slug.replace(/[\s-]+/g, "-")

	return slug
}

export function slugifyReverse(input) {
	if (!input) return ""

	const enToRu = {
		shch: "щ",
		yo: "ё",
		zh: "ж",
		ch: "ч",
		sh: "ш",
		yu: "ю",
		ya: "я",
		kh: "х",
		ts: "ц",
		"y`": "ы",
		"e`": "э",
		"`": "ъ",
		"'": "ь",
		a: "а",
		b: "б",
		v: "в",
		g: "г",
		d: "д",
		e: "е",
		z: "з",
		i: "и",
		k: "к",
		l: "л",
		m: "м",
		n: "н",
		o: "о",
		p: "п",
		r: "р",
		s: "с",
		t: "т",
		u: "у",
		f: "ф",
		y: "й",
	}

	const words = input.split("-")

	const translatedWords = words.map(word => {
		let translated = ""
		let i = 0
		while (i < word.length) {
			let found = false

			for (let len = 4; len > 0; len--) {
				const part = word.slice(i, i + len)
				if (enToRu[part]) {
					translated += enToRu[part]
					i += len
					found = true
					break
				}
			}

			if (!found) {
				translated += word[i]
				i++
			}
		}
		return translated
	})

	return translatedWords.join(" ")
}

```

## Как их использовать.
**В проекте: internet-aiz:**
- /pages/[tag].astro
- /components/Tags.astro
- и прочее...