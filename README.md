# Way-of-Waqt
Plan your day around the 5 daily Salah. A time management app for Muslims.
To test your prayer time application in **GitHub Codespaces**, you'll need to set up a Codespace environment, add the fixed code, install dependencies, and run the app and tests. Below are step-by-step instructions to add the code to a GitHub repository, configure Codespaces, and test the application.

---

### Step 1: Set Up a GitHub Repository
1. **Create a New Repository**:
   - Go to [GitHub](https://github.com) and click **New repository**.
   - Name it (e.g., `prayer-time-app`).
   - Initialize with a `README.md` and select the **Node** `.gitignore` template.
   - Click **Create repository**.

2. **Clone the Repository Locally** (Optional, if editing locally first):
   - Clone the repo:
     ```bash
     git clone https://github.com/your-username/prayer-time-app.git
     cd prayer-time-app
     ```
   - Alternatively, you can add files directly in Codespaces (see below).

---

### Step 2: Create a GitHub Codespace
1. **Open Codespaces**:
   - Go to your repository on GitHub.
   - Click the **Code** button (green button) and select the **Codespaces** tab.
   - Click **Create codespace on main** to start a new Codespace.

2. **Wait for Setup**:
   - Codespaces will set up a cloud-based development environment with a default configuration (VS Code in the browser).
   - This may take a few minutes the first time.

---

### Step 3: Add the Code to Codespaces
1. **Create Project Structure**:
   - In the Codespaces terminal (or VS Code explorer), create the following directory structure:
     ```
     src/
       ├── components/
       │   ├── PrayerTimeDisplay.tsx
       │   ├── TimeFormatToggle.tsx
       │   ├── DatePicker.tsx
       │   └── useLocation.ts
       ├── App.tsx
       ├── index.tsx
       └── index.css
     ```

2. **Copy the Fixed Code**:
   - Use the fixed code from my previous response. For each file, open it in the Codespaces editor and paste the content:
     - `index.tsx`:
       ```tsx
       import React from 'react';
       import { createRoot } from 'react-dom/client';
       import './index.css';
       import App from './App';

       const root = createRoot(document.getElementById('root')!);
       root.render(
         <React.StrictMode>
           <App />
         </React.StrictMode>
       );
       ```
     - `App.tsx`:
       ```tsx
       import React, { useState } from 'react';
       import PrayerTimeDisplay from './components/PrayerTimeDisplay';
       import TimeFormatToggle from './components/TimeFormatToggle';
       import DatePicker from './components/DatePicker';

       interface AppProps {}

       const App: React.FC<AppProps> = () => {
         const [formatType, setFormatType] = useState<'12' | '24'>('24');
         const [selectedDate, setSelectedDate] = useState<Date>(new Date());

         return (
           <div className="container">
             <header>
               <h1>Way of Waqt</h1>
             </header>
             <main>
               <TimeFormatToggle formatType={formatType} setFormatType={setFormatType} />
               <DatePicker onDateChange={setSelectedDate} />
               <PrayerTimeDisplay formatType={formatType} selectedDate={selectedDate} />
             </main>
           </div>
         );
       };

       export default App;
       ```
     - `PrayerTimeDisplay.tsx`:
       ```tsx
       import React, { useEffect, useState } from 'react';
       import { PrayerTimes, Coordinates } from 'adhan';
       import { useLocation } from './useLocation';
       import { format } from 'date-fns';

       interface PrayerTimeDisplayProps {
         formatType: '12' | '24';
         selectedDate?: Date;
       }

       const PrayerTimeDisplay: React.FC<PrayerTimeDisplayProps> = ({ formatType, selectedDate = new Date() }) => {
         const { location, error: locationError } = useLocation();
         const [prayerTimes, setPrayerTimes] = useState<PrayerTimes | null>(null);
         const [error, setError] = useState<string | null>(null);

         useEffect(() => {
           if (location) {
             try {
               const coordinates = new Coordinates(location.latitude, location.longitude);
               const params = {
                 madhab: 'Shafi',
                 highLatitudeRule: 'MiddleOfTheNight',
               };
               const prayerTimes = new PrayerTimes(coordinates, selectedDate, params);
               setPrayerTimes(prayerTimes);
               setError(null);
             } catch (err) {
               setError('Failed to calculate prayer times');
               console.error(err);
             }
           } else if (locationError) {
             setError(locationError);
           }
         }, [location, locationError, selectedDate]);

         const prayerKeys = ['fajr', 'sunrise', 'dhuhr', 'asr', 'maghrib', 'isha'];

         return (
           <div>
             <h2>Prayer Times</h2>
             {error ? (
               <p className="error">{error}</p>
             ) : !location || !prayerTimes ? (
               <p>Loading...</p>
             ) : (
               <ul>
                 {prayerKeys.map((key) => (
                   <li key={key}>
                     {key.charAt(0).toUpperCase() + key.slice(1)}:{' '}
                     {formatTime(prayerTimes[key as keyof PrayerTimes] as Date, formatType)}
                   </li>
                 ))}
               </ul>
             )}
           </div>
         );
       };

       const formatTime = (time: Date, formatType: '12' | '24'): string => {
         return format(time, formatType === '12' ? 'hh:mm a' : 'HH:mm');
       };

       export default PrayerTimeDisplay;
       ```
     - `TimeFormatToggle.tsx`:
       ```tsx
       import React from 'react';

       interface TimeFormatToggleProps {
         formatType: '12' | '24';
         setFormatType: (type: '12' | '24') => void;
       }

       const TimeFormatToggle: React.FC<TimeFormatToggleProps> = ({ formatType, setFormatType }) => {
         return (
           <div className="mb-4 flex items-center space-x-2" role="radiogroup" aria-label="Time format selector">
             <span className="text-sm font-medium">Time Format:</span>
             <button
               onClick={() => setFormatType('12')}
               className={`px-2 py-1 rounded text-sm ${formatType === '12' ? 'bg-blue-500 text-white' : 'bg-gray-100'}`}
               aria-checked={formatType === '12'}
               role="radio"
             >
               12h
             </button>
             <button
               onClick={() => setFormatType('24')}
               className={`px-2 py-1 rounded text-sm ${formatType === '24' ? 'bg-blue-500 text-white' : 'bg-gray-100'}`}
               aria-checked={formatType === '24'}
               role="radio"
             >
               24h
             </button>
           </div>
         );
       };

       export default TimeFormatToggle;
       ```
     - `DatePicker.tsx`:
       ```tsx
       import React, { useState } from 'react';

       interface DatePickerProps {
         onDateChange: (date: Date) => void;
       }

       const DatePicker: React.FC<DatePickerProps> = ({ onDateChange }) => {
         const [selectedDate, setSelectedDate] = useState<string>(new Date().toISOString().split('T')[0]);

         const handleDateChange = (e: React.ChangeEvent<HTMLInputElement>) => {
           const newDate = e.target.value;
           setSelectedDate(newDate);
           const parsedDate = new Date(newDate);
           if (!isNaN(parsedDate.getTime())) {
             onDateChange(parsedDate);
           }
         };

         return (
           <div className="mb-4">
             <label htmlFor="date" className="text-sm font-medium">
               Select a Date:
             </label>
             <input
               type="date"
               id="date"
               value={selectedDate}
               onChange={handleDateChange}
               className="ml-2 p-1 border rounded"
             />
           </div>
         );
       };

       export default DatePicker;
       ```
     - `useLocation.ts`:
       ```ts
       import { useState, useEffect } from 'react';

       interface Location {
         latitude: number;
         longitude: number;
       }

       export const useLocation = () => {
         const [location, setLocation] = useState<Location | null>(null);
         const [error, setError] = useState<string | null>(null);

         useEffect(() => {
           if (!navigator.geolocation) {
             setError('Geolocation is not supported by your browser');
             return;
           }

           navigator.geolocation.getCurrentPosition(
             (position) => {
               setLocation({
                 latitude: position.coords.latitude,
                 longitude: position.coords.longitude,
               });
               setError(null);
             },
             (error) => {
               setError('Unable to retrieve location. Please allow location access.');
               console.error('Error getting location', error);
             }
           );
         }, []);

         return { location, error };
       };
       ```
     - `index.css` (basic styling, adjust as needed):
       ```css
       body {
         font-family: Arial, sans-serif;
         margin: 0;
         padding: 0;
       }
       .container {
         max-width: 600px;
         margin: 0 auto;
         padding: 20px;
       }
       .error {
         color: red;
       }
       ```

3. **Add `package.json`**:
   - Create a `package.json` file with the necessary dependencies and scripts:
     ```json
     {
       "name": "prayer-time-app",
       "version": "1.0.0",
       "private": true,
       "dependencies": {
         "react": "^18.2.0",
         "react-dom": "^18.2.0",
         "adhan": "^4.2.0",
         "date-fns": "^3.6.0",
         "typescript": "^5.5.3"
       },
       "devDependencies": {
         "@testing-library/react": "^16.0.0",
         "@testing-library/jest-dom": "^6.4.6",
         "jest": "^29.7.0",
         "ts-jest": "^29.2.4",
         "eslint": "^8.57.0",
         "@typescript-eslint/parser": "^7.18.0",
         "@typescript-eslint/eslint-plugin": "^7.18.0",
         "identity-obj-proxy": "^3.0.0",
         "react-scripts": "^5.0.1"
       },
       "scripts": {
         "start": "react-scripts start",
         "build": "react-scripts build",
         "test": "jest",
         "lint": "eslint src/**/*.{ts,tsx}"
       },
       "browserslist": {
         "production": [">0.2%", "not dead", "not op_mini all"],
         "development": ["last 1 chrome version", "last 1 firefox version", "last 1 safari version"]
       }
     }
     ```

4. **Add TypeScript Configuration** (`tsconfig.json`):
   ```json
   {
     "compilerOptions": {
       "target": "es5",
       "lib": ["dom", "dom.iterable", "esnext"],
       "allowJs": true,
       "skipLibCheck": true,
       "esModuleInterop": true,
       "allowSyntheticDefaultImports": true,
       "strict": true,
       "forceConsistentCasingInFileNames": true,
       "noFallthroughCasesInSwitch": true,
       "module": "esnext",
       "moduleResolution": "node",
       "resolveJsonModule": true,
       "isolatedModules": true,
       "noEmit": true,
       "jsx": "react-jsx"
     },
     "include": ["src"]
   }
   ```

5. **Add Jest Configuration** (`jest.config.js`):
   ```js
   module.exports = {
     preset: 'ts-jest',
     testEnvironment: 'jsdom',
     setupFilesAfterEnv: ['@testing-library/jest-dom'],
     moduleNameMapper: {
       '\\.(css|less)$': 'identity-obj-proxy'
     }
   };
   ```

6. **Add ESLint Configuration** (`.eslintrc.json`):
   ```json
   {
     "env": {
       "browser": true,
       "es2021": true,
       "jest": true
     },
     "extends": [
       "eslint:recommended",
       "plugin:@typescript-eslint/recommended",
       "plugin:react/recommended"
     ],
     "parser": "@typescript-eslint/parser",
     "parserOptions": {
       "ecmaVersion": 12,
       "sourceType": "module"
     },
     "plugins": ["@typescript-eslint", "react"],
     "rules": {
       "react/prop-types": "off"
     }
   }
   ```

7. **Add Unit Tests** (e.g., `src/components/PrayerTimeDisplay.test.tsx`):
   ```tsx
   import React from 'react';
   import { render, screen } from '@testing-library/react';
   import PrayerTimeDisplay from './PrayerTimeDisplay';
   import { useLocation } from './useLocation';

   jest.mock('./useLocation');

   describe('PrayerTimeDisplay', () => {
     beforeEach(() => {
       (useLocation as jest.Mock).mockReturnValue({
         location: { latitude: 51.5074, longitude: -0.1278 },
         error: null
       });
     });

     it('renders loading state when location is not available', () => {
       (useLocation as jest.Mock).mockReturnValue({ location: null, error: null });
       render(<PrayerTimeDisplay formatType="24" />);
       expect(screen.getByText('Loading...')).toBeInTheDocument();
     });

     it('renders prayer times when location is available', () => {
       render(<PrayerTimeDisplay formatType="24" />);
       expect(screen.getByText('Prayer Times')).toBeInTheDocument();
       expect(screen.getByText(/Fajr:/i)).toBeInTheDocument();
     });

     it('displays error when location fails', () => {
       (useLocation as jest.Mock).mockReturnValue({
         location: null,
         error: 'Unable to retrieve location'
       });
       render(<PrayerTimeDisplay formatType="24" />);
       expect(screen.getByText('Unable to retrieve location')).toBeInTheDocument();
     });
   });
   ```

8. **Add `public/index.html`**:
   - Create a `public` folder and add `index.html`:
     ```html
     <!DOCTYPE html>
     <html lang="en">
       <head>
         <meta charset="utf-8" />
         <meta name="viewport" content="width=device-width, initial-scale=1" />
         <title>Prayer Time App</title>
       </head>
       <body>
         <div id="root"></div>
       </body>
     </html>
     ```

---

### Step 4: Install Dependencies in Codespaces
1. **Open the Terminal**:
   - In Codespaces, use the integrated terminal (Ctrl+` or Terminal > New Terminal).

2. **Install Dependencies**:
   ```bash
   npm install
   ```
   - This installs all dependencies listed in `package.json`.

---

### Step 5: Test the Application in Codespaces
1. **Run the Development Server**:
   ```bash
   npm start
   ```
   - Codespaces will start the React app on `http://localhost:3000`.
   - You’ll see a prompt to open the app in a browser or a forwarded port. Click **Open in Browser** to view the app.
   - **Note**: Geolocation may not work in Codespaces unless you mock it or deploy the app to a public URL (see Step 7). For testing, you can temporarily modify `useLocation.ts` to return a fixed location:
     ```ts
     export const useLocation = () => {
       return {
         location: { latitude: 51.5074, longitude: -0.1278 }, // London
         error: null
       };
     };
     ```

2. **Manually Test the App**:
   - Verify the app loads with the “Way of Waqt” header.
   - Toggle between 12h and 24h formats and check if prayer times update.
   - Select a different date and confirm prayer times reflect the new date.
   - Check error handling by denying geolocation (or using the mocked error state).

3. **Run Linting**:
   ```bash
   npm run lint
   ```
   - Fix any ESLint errors reported.

4. **Run Type-Checking**:
   ```bash
   npx tsc --noEmit
   ```
   - Ensure no TypeScript errors.

5. **Run Unit Tests**:
   ```bash
   npm test
   ```
   - Verify that the tests pass. If you added more tests for other components, ensure they cover key functionality.

---

### Step 6: Commit and Push Changes
1. **Stage and Commit Files**:
   - In the Codespaces VS Code interface, go to the **Source Control** tab (or use the terminal):
     ```bash
     git add .
     git commit -m "Add prayer time app with tests"
     ```

2. **Push to GitHub**:
   ```bash
   git push origin main
   ```

3. **Verify on GitHub**:
   - Go to your repository on GitHub and confirm the files are present.

---

### Step 7: Optional - Test Geolocation and Deploy
**Geolocation Note**:
- Geolocation (`navigator.geolocation`) requires a secure context (HTTPS or localhost). Codespaces runs on `localhost`, but the browser may block geolocation requests in a cloud environment.
- To test geolocation properly:
  - Deploy the app to a platform like Vercel or Netlify (see below).
  - Or, continue using the mocked location for development.

**Deploy to Vercel for Testing**:
1. Install the Vercel CLI in Codespaces:
   ```bash
   npm install -g vercel
   ```
2. Deploy the app:
   ```bash
   vercel
   ```
   - Follow the prompts to log in, link the project, and deploy.
   - Vercel will provide a public URL (e.g., `https://prayer-time-app.vercel.app`).
3. Test the deployed app:
   - Open the URL in a browser.
   - Allow geolocation when prompted and verify prayer times load.
   - Test date selection and time format toggling.

**Add GitHub Actions for CI** (Optional):
- To automate testing on every push, add a GitHub Actions workflow (as described in my previous response):
  - Create `.github/workflows/ci.yml`:
    ```yaml
    name: CI

    on:
      push:
        branches: [main]
      pull_request:
        branches: [main]

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v4
          - name: Set up Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: 'npm'

          - name: Install dependencies
            run: npm ci

          - name: Run linting
            run: npm run lint

          - name: Run type-checking
            run: npx tsc --noEmit

          - name: Run tests
            run: npm test
    ```
- Commit and push this file, then check the **Actions** tab on GitHub to ensure the workflow runs successfully.

---

### Step 8: Troubleshooting
- **Geolocation Issues**:
  - If geolocation fails in Codespaces, use the mocked location or deploy to Vercel.
  - Ensure the browser allows location access for the deployed URL.
- **Dependency Errors**:
  - If `npm install` fails, check `package.json` for typos or run `npm cache clean --force` and retry.
- **Test Failures**:
  - Check test error messages and ensure mocks (e.g., `useLocation`) are set up correctly.
- **Port Forwarding in Codespaces**:
  - If `npm start` doesn’t prompt to open the app, go to the **Ports** tab in Codespaces, find port `3000`, and set it to **Public** visibility.

---

### Summary
- **Codespaces Setup**: Created a Codespace, added code files, and configured `package.json`, TypeScript, Jest, and ESLint.
- **Testing**: Ran the app (`npm start`), linting (`npm run lint`), type-checking (`npx tsc --noEmit`), and unit tests (`npm test`).
- **Geolocation**: Mocked location for development; deploy to Vercel for real geolocation testing.
- **GitHub Integration**: Committed and pushed code, with optional GitHub Actions for CI.

If you run into specific errors (e.g., dependency issues, test failures, or geolocation problems), share the details, and I’ll provide targeted fixes. Let me know if you need additional tests or deployment help!
