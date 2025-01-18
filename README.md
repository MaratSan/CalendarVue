# Calendar Component (Vue 3)

## Overview
This is a reusable Vue 3 component for displaying two months of a calendar side by side. The calendar allows users to select date ranges, navigate between months, and choose predefined favorite terms.

## Features
- Displays two months side by side.
- Allows navigation between months.
- Supports date range selection.
- Highlights the current date.
- Customizable favorite terms.
- Interactive hover effects for better user experience.

## Installation
1. Clone the repository or copy the component files into your project.
2. Install necessary dependencies:
   ```bash
   npm install
   ```
3. Import the component into your Vue project.

## Usage
### Template
```vue
<template>
  <Calendar />
</template>
```

### Script
```vue
<script setup>
import Calendar from './components/Calendar.vue';
</script>
```

## Props
The component does not currently accept any props but can be extended for customization, such as:
- `initialMonth` (Number): The starting month to display.
- `favoriteTerms` (Array): Custom favorite terms.

## Methods
- **prevMonth**: Navigates to the previous month.
- **nextMonthHandler**: Navigates to the next month.
- **selectDay(day)**: Selects a specific day.
- **selectFavorite(range)**: Selects a predefined range from favorite terms.

## Computed Properties
- `currentMonthYear`: Displays the current month and year in a readable format.
- `nextMonthYear`: Displays the next month and year in a readable format.
- `daysInMonth`: Computes the days for the current month.
- `nextMonthDays`: Computes the days for the next month.

## Customization
To customize the styles, modify the scoped CSS in the component. Key CSS classes include:
- `.btn`: For navigation buttons.
- `.calendar-body`: For the main calendar container.
- `.calendar-day:hover`: For hover effects on individual days.

## Example
```vue
<template>
  <Calendar />
</template>

<script setup>
import Calendar from './Calendar.vue';
</script>
```

## Future Enhancements
- Add support for multiple languages (i18n).
- Accept configuration props for greater flexibility.
- Add unit tests for better reliability.

## License
This project is open-source and available under the MIT License.
