<script setup>
import { ref, computed } from "vue"
const props = defineProps({
  rezumes: Array,
  reviews: Array,
})
const emit = defineEmits(['actionSix'])

const minRating = ref(0)
const filterGenre = ref('all')

function addToCart(book) {
  emit('actionSix', book)
}

const staticResumes = ref([
  {
    title: 'Коралина',
    pay: '500р.',
    genre: ['детская фантастика ', 'триллер '],
    description: '«Коралина» — это история о любопытной и отважной девочке, которая находит потайную дверь в свой дом, ведущую в альтернативный мир, где её ждут другая, идеальная, но зловещая версия её родителей. Книга Нила Геймана, по которой был снят мультфильм, учит ценить настоящую реальность и свою семью, несмотря на все её несовершенства.',
    author: 'Нил Гейман',
    cover: 'твердая',
    rating: 4.2,
    image: 'Coralina.jpg'
  },
  {
    title: 'Маленькая хозяйка большого дома',
    pay: '350р.',
    genre: ['роман ', 'приключенческая художественная литература '],
    description: 'Сюжет романа Джека Лондона «Маленькая хозяйка большого дома» вращается вокруг любовного треугольника между Паолой, её мужем Диком и их давним другом Ивэном. Паола и Дик – успешная, но занятая пара, хозяйка поместья и заводчик лошадей. Когда Ивэн появляется в их доме, он влюбляется в Паолу, которая обнаруживает, что её чувства взаимны. В итоге Паола оказывается перед сложным выбором между преданностью мужу и страстью к Ивэну, что приводит к трагическому финалу.',
    author: 'Джек Лондон',
    cover: 'мягкая',
    rating: 3.7,
    image: 'little.jpg'
  },
  {
    title: 'Рик и Морти',
    pay: '550р.',
    genre: ['научно-популярная литература '],
    description: 'Из книги вы узнаете, как мы можем использовать темную материю и энергию, что такое биохакинг, возможно ли контролировать нервную систему таракана при помощи языка и многое другое.',
    author: 'Мэтт Брэди',
    cover: 'мягкая',
    rating: 4.1,
    image: 'rick.jpg'
  },
  {
    title: 'Последние дни Помпеи',
    pay: '400р.',
    genre: ['роман ', 'исторический '],
    description: '«Последние дни Помпеи» — это исторический роман английского писателя Эдварда Бульвер-Литтона, опубликованный в 1834 году и вдохновленный одноименной картиной Карла Брюллова. В основе сюжета — любовная история афинского юноши Главка и гречанки Ионы, которая происходит на фоне надвигающейся катастрофы — извержения Везувия, уничтожившего Помпеи в 79 году н.э. Их счастью мешает египетский жрец Арбак, который также влюблен в Иону и жаждет отомстить Главкам, но все герои оказываются перед лицом гораздо более страшной угрозы — извержения вулкана.',
    author: 'Эдвард Бульвер-Литтон',
    cover: 'мягкая',
    rating: 4.9,
    image: 'pompei.jpg'
  },
  {
    title: 'Математика с дурацкими рисунками',
    pay: '1000р.',
    genre: ['научно-популярная литература '],
    description: '«Математика с дурацкими рисунками» — это книга Бена Орлина, которая объясняет сложные математические концепции через забавные иллюстрации, истории и игры, делая математику доступной и увлекательной. Вместо формальных учебников, она предлагает идеи и игры, которые показывают связь математики с повседневной жизнью, используя логику, пространственное и стратегическое мышление.',
    author: 'Бен Орлин',
    cover: 'твердая',
    rating: 3.0,
    image: 'matematick.jpg'
  },
  {
    title: '451° по Фаренгейту',
    pay: '600р.',
    genre: ['антиутопия '],
    description: 'Книга описывает общество будущего, где книги запрещены и подлежат уничтожению отрядами пожарных, а люди развлечены телеэкранами и поверхностным контентом. Сюжет рассказывает историю пожарного Гая Монтэга, который начинает сомневаться в своей работе и ценностях общества после встречи с девушкой Клариссой.',
    author: 'Рэй Бредбери',
    cover: 'твердая',
    rating: 5.0,
    image: '451.jpg'
  },
])

const allGenres = computed(() => {
  const genres = new Set

  staticResumes.value.forEach(book => {
    if (Array.isArray(book.genre)) {
      book.genre.forEach(g => genres.add(g.trim()))
    } else if (book.genre) {
      genres.add(book.genre.trim)
    }
  })

  if (props.rezumes && Array.isArray(props.rezumes)) {
    props.rezumes.forEach(book => {
      if (Array.isArray(book.genre)) {
        book.genre.forEach(g => genres.add(g.trim()))
      } else if (book.genre) {
        genres.add(book.genre.trim())
      }
    })
  }

  return ['all', ...Array.from(genres).sort()]
})

const resumesWithRating = computed(() => {
  const dynamicResumes = (props.rezumes && Array.isArray(props.rezumes)) 
    ? props.rezumes.map(resume => {
        const copywriterReviews = props.reviews?.filter(r => r.copywriter === resume.author) || []
        const rating = copywriterReviews.length > 0
          ? (copywriterReviews.reduce((sum, r) => sum + r.rating, 0) / copywriterReviews.length).toFixed(1)
          : null

        return {
          ...resume,
          rating,
          hasPortfolio: resume.portfolio && resume.portfolio.trim() !== '' && resume.portfolio.toLowerCase() !== 'нет'
        }
      })
    : []

  return [...staticResumes.value, ...dynamicResumes]
})

const filteredResumes = computed(() => {
if (!resumesWithRating.value || !Array.isArray(resumesWithRating.value)) {
    return []
  }
  
  return resumesWithRating.value.filter(book => {

    const genreMatch =
      filterGenre.value === 'all' ||
      (Array.isArray(book.genre) && book.genre.includes(filterGenre.value)) ||
      book.genre === filterGenre.value

    const ratingMatch =
      minRating.value === 0 ||
      (book.rating && parseFloat(book.rating) >= minRating.value)

    return ratingMatch && genreMatch
  })
})

function getImageUrl(imageName) {
  if (!imageName) return ''
  return new URL(`../assets/${imageName}`, import.meta.url).href
}
</script>
<template>
  <div class="filters">
    <div class="filter-group">
      <label>Жанр:</label>
      <select v-model="filterGenre">
        <option v-for="genre in allGenres" :key="genre" :value="genre">
          {{ genre === 'all' ? 'Все жанры' : genre }}
        </option>
      </select>
    </div>

    <div class="filter-group">
      <label>Минимальный рейтинг:</label>
      <select v-model="minRating">
        <option :value="0">Любой</option>
        <option :value="3">3+</option>
        <option :value="4">4+</option>
        <option :value="4.5">4.5+</option>
      </select>
    </div>
  </div>

  <div class="gridd">
    <div v-for="(item, index) in filteredResumes" :key="index" class="square">
      <div class="card-content">
        <div class="book-image-container">
          <img
          :src="getImageUrl(item.image)" 
          :alt="item.title"
          class="book-image"
          @error="handleImageError">
        </div>
        <p>Название: {{ item.title }}</p>
        <div class="genres-container">
          <span class="genre-label">Жанры:</span>
          <div class="genres-list">
            <span v-for="(genre, genreIndex) in (Array.isArray(item.genre) ? item.genre : [item.genre])"
              :key="genreIndex" class="genre-tag">
              {{ genre }}
            </span>
          </div>
        </div>
        <p>Цена: {{ item.pay }}</p>
        <details class="description-dropdown">
          <summary>Описание</summary>
          <div class="description-content">
            {{ item.description }}
          </div>
        </details>
        <p>Автор: {{ item.author }}</p>
        <div class="rating-container">
          <p v-if="item.rating !== null">Рейтинг: {{ item.rating }} ★</p>
          <p v-else>Рейтинг: нет оценок</p>
        </div>
      </div>
      <button @click="addToCart(item)" class="btn">В корзину</button>
    </div>
  </div>
</template>
<style scoped>
.gridd {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  padding: 20px;
  align-items: start;
}

.square {
  position: relative;
  width: 300px;
  min-height: 400px;
  height: auto;
  color: black;
  background: linear-gradient(to bottom left, rgba(183, 84, 45, 0.5), rgba(250, 231, 167, 0.5));
  border-radius: 5%;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-top: 50px;
  word-wrap: break-word;
  white-space: normal;
  transition: height 0.3s ease;
}

.card-content {
  flex-grow: 1;
}

.description-dropdown {
  margin: 10px 0;
  position: relative;
}

.description-dropdown summary {
  cursor: pointer;
  font-weight: bold;
  padding: 8px 12px;
  background: rgba(163, 72, 33, 0.1);
  border-radius: 4px;
  list-style: none;
  border: 1px solid rgba(163, 72, 33, 0.3);
  transition: background-color 0.3s;
}

.description-dropdown summary:hover {
  background: rgba(163, 72, 33, 0.2);
}

.description-dropdown summary::-webkit-details-marker {
  display: none;
}

.description-dropdown[open] summary {
  border-radius: 4px 4px 0 0;
  margin-bottom: 0;
}

.description-content {
  padding: 12px;
  background: rgba(197, 68, 43, 0.3);
  border-radius: 0 0 4px 4px;
  border: 1px solid rgba(163, 72, 33, 0.3);
  border-top: none;
  margin-top: -1px;
  max-height: 200px;
  overflow-y: auto;
}

.rating-container {
  margin: 15px 0;
}

.filters {
  display: flex;

  gap: 20px;
  margin: 20px 0;
  margin-top: 100px;
  padding: 15px;
  background: rgba(193, 87, 33, 0.2);
  border-radius: 10px;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.filter-group label {
  font-weight: bold;
}

.filter-group select {
  padding: 5px 10px;
  border-radius: 5px;
  border: 1px solid #845b49;
}

.square p:last-of-type {
  margin-bottom: 50px;
}

.btn {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: calc(100% - 100px);
  margin-top: 10px;
  height: auto;
  background-color: #a34821;
}

.btn:hover {
  background-color: #bf6138;
}

.book-image-container {
  text-align: center;
  margin-bottom: 15px;
}

.book-image {
  max-width: 150px;
  max-height: 200px;
  width: auto;
  height: auto;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  object-fit: cover;
}

.book-title {
  font-weight: bold;
  font-size: 1.2em;
  margin-bottom: 10px;
  text-align: center;
}
</style>