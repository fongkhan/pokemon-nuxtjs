# Pokemon Finder Application

A modern web application built with Nuxt.js that allows users to search and view detailed information about Pokemon. The application uses the PokeAPI to fetch Pokemon data and presents it in a clean, responsive interface.

## Features

- **Pokemon Search**: Search for Pokemon by name or ID
- **Detailed Pokemon Information**:
  - Official artwork and sprites
  - Type information with color coding
  - Base stats with visual bars
  - Physical characteristics (height, weight)
  - Abilities (including hidden abilities)
  - Pokemon descriptions in English
- **Responsive Design**: Works seamlessly on desktop and mobile devices
- **Real-time Updates**: Automatic data refresh when searching for the same Pokemon

## Prerequisites

- Node.js (Latest LTS version recommended)
- NPM, Yarn, PNPM, or Bun package manager

## Setup

Clone the repository and install dependencies:

```bash
# npm
npm install

# pnpm
pnpm install

# yarn
yarn install

# bun
bun install
```

## Development Server

Start the development server on `http://localhost:3000`:

```bash
# npm
npm run dev

# pnpm
pnpm dev

# yarn
yarn dev

# bun
bun run dev
```

## Usage

1. Open the application in your browser
2. Enter a Pokemon name or ID in the search box
3. Press Enter or click the Search button
4. View detailed Pokemon information including:
   - Basic stats and characteristics
   - Type information
   - Abilities
   - Official artwork

## Production Deployment

Build the application for production:

```bash
# npm
npm run build

# pnpm
pnpm build

# yarn
yarn build

# bun
bun run build
```

Preview the production build locally:

```bash
# npm
npm run preview

# pnpm
pnpm preview

# yarn
yarn preview

# bun
bun run preview
```

## Technologies Used

- [Nuxt.js](https://nuxt.com/) - The Vue.js Framework
- [PokeAPI](https://pokeapi.co/) - Pokemon Data API
- TypeScript - For type safety
- Vue 3 Composition API - For state management

## Credits

- Pokemon data provided by [PokeAPI](https://pokeapi.co/)
- Pokemon is a trademark of Nintendo/Creatures Inc./GAME FREAK inc.

For more information about deployment, check out the [Nuxt deployment documentation](https://nuxt.com/docs/getting-started/deployment).
