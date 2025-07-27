# Claude Generated Code

**Task**: Objective: Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code  Claude Tasks:  🧩 Feature Implementation Read the existing codebase and integrate new feature modules where appropriate.  Ensure new code follows existing patterns and conventions.  🗂️ Example Features (replace/add specifics as needed): Add user role-based access control  Implement export to PDF  Add multi-language support  Enable dark mode toggle  🧪 Testing & Documentation Add unit/integration tests for new functionality.  Update README or in-code comments if required.

**Generated on**: Sun Jul 27 15:24:20 UTC 2025

## Generated Content

```
I'll help you implement new features for this government billing/invoicing React app. Based on the current codebase, I'll implement several key features that would benefit this application.

## FILES_TO_CREATE

1. `src/components/UserRoleGuard.tsx`
2. `src/services/AuthService.ts`
3. `src/services/PdfService.ts`
4. `src/services/I18nService.ts`
5. `src/hooks/useAuth.ts`
6. `src/hooks/useDarkMode.ts`
7. `src/hooks/useI18n.ts`
8. `src/context/AuthContext.tsx`
9. `src/context/ThemeContext.tsx`
10. `src/context/I18nContext.tsx`
11. `src/utils/validation.ts`
12. `src/translations/en.json`
13. `src/translations/es.json`
14. `src/components/PDFExport/PDFExport.tsx`
15. `src/components/ThemeToggle/ThemeToggle.tsx`
16. `src/components/LanguageSelector/LanguageSelector.tsx`
17. `src/components/UserRoleGuard/UserRoleGuard.tsx`

## FILES_TO_MODIFY

1. `src/App/App.js` (convert to TypeScript)
2. `src/index.js` (convert to TypeScript)
3. `package.json`

## CODE_CHANGES

### 1. New File: `src/types/auth.ts`
```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  permissions: Permission[];
}

export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user',
  VIEWER = 'viewer'
}

export enum Permission {
  CREATE_INVOICE = 'create_invoice',
  EDIT_INVOICE = 'edit_invoice',
  DELETE_INVOICE = 'delete_invoice',
  EXPORT_PDF = 'export_pdf',
  VIEW_ALL_INVOICES = 'view_all_invoices',
  MANAGE_USERS = 'manage_users'
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}
```

### 2. New File: `src/context/AuthContext.tsx`
```typescript
import React, { createContext, useContext, useReducer, ReactNode } from 'react';
import { User, AuthState, UserRole, Permission } from '../types/auth';

interface AuthContextType extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  hasPermission: (permission: Permission) => boolean;
  hasRole: (role: UserRole | UserRole[]) => boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_ERROR'; payload: string }
  | { type: 'LOGOUT' };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        isLoading: false,
        isAuthenticated: true,
        user: action.payload,
        error: null
      };
    case 'LOGIN_ERROR':
      return {
        ...state,
        isLoading: false,
        isAuthenticated: false,
        user: null,
        error: action.payload
      };
    case 'LOGOUT':
      return {
        ...state,
        isAuthenticated: false,
        user: null,
        error: null
      };
    default:
      return state;
  }
};

interface AuthProviderProps {
  children: ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const initialState: AuthState = {
    user: null,
    isAuthenticated: false,
    isLoading: false,
    error: null
  };

  const [state, dispatch] = useReducer(authReducer, initialState);

  const login = async (email: string, password: string): Promise<void> => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      // Simulate API call - replace with actual authentication
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // Mock user based on email
      const mockUser: User = {
        id: '1',
        email,
        name: email.split('@')[0],
        role: email.includes('admin') ? UserRole.ADMIN : UserRole.USER,
        permissions: email.includes('admin') 
          ? Object.values(Permission)
          : [Permission.CREATE_INVOICE, Permission.EDIT_INVOICE, Permission.EXPORT_PDF]
      };

      localStorage.setItem('authToken', 'mock-jwt-token');
      dispatch({ type: 'LOGIN_SUCCESS', payload: mockUser });
    } catch (error) {
      dispatch({ type: 'LOGIN_ERROR', payload: 'Login failed' });
    }
  };

  const logout = (): void => {
    localStorage.removeItem('authToken');
    dispatch({ type: 'LOGOUT' });
  };

  const hasPermission = (permission: Permission): boolean => {
    return state.user?.permissions.includes(permission) || false;
  };

  const hasRole = (role: UserRole | UserRole[]): boolean => {
    if (!state.user) return false;
    const roles = Array.isArray(role) ? role : [role];
    return roles.includes(state.user.role);
  };

  return (
    <AuthContext.Provider value={{
      ...state,
      login,
      logout,
      hasPermission,
      hasRole
    }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### 3. New File: `src/context/ThemeContext.tsx`
```typescript
import React, { createContext, useContext, useEffect, useState, ReactNode } from 'react';

interface ThemeContextType {
  isDarkMode: boolean;
  toggleDarkMode: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

interface ThemeProviderProps {
  children: ReactNode;
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [isDarkMode, setIsDarkMode] = useState<boolean>(() => {
    const saved = localStorage.getItem('darkMode');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    return saved ? JSON.parse(saved) : prefersDark;
  });

  useEffect(() => {
    document.body.classList.toggle('dark', isDarkMode);
    localStorage.setItem('darkMode', JSON.stringify(isDarkMode));
  }, [isDarkMode]);

  const toggleDarkMode = () => {
    setIsDarkMode(prev => !prev);
  };

  return (
    <ThemeContext.Provider value={{ isDarkMode, toggleDarkMode }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 4. New File: `src/context/I18nContext.tsx`
```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

export type Language = 'en' | 'es';

interface I18nContextType {
  language: Language;
  setLanguage: (lang: Language) => void;
  t: (key: string) => string;
}

const I18nContext = createContext<I18nContextType | undefined>(undefined);

// Import translations
import en from '../translations/en.json';
import es from '../translations/es.json';

const translations = { en, es };

interface I18nProviderProps {
  children: ReactNode;
}

export const I18nProvider: React.FC<I18nProviderProps> = ({ children }) => {
  const [language, setLanguage] = useState<Language>(() => {
    const saved = localStorage.getItem('language') as Language;
    return saved || 'en';
  });

  const handleSetLanguage = (lang: Language) => {
    setLanguage(lang);
    localStorage.setItem('language', lang);
  };

  const t = (key: string): string => {
    const keys = key.split('.');
    let value: any = translations[language];
    
    for (const k of keys) {
      value = value?.[k];
    }
    
    return value || key;
  };

  return (
    <I18nContext.Provider value={{ language, setLanguage: handleSetLanguage, t }}>
      {children}
    </I18nContext.Provider>
  );
};

export const useI18n = (): I18nContextType => {
  const context = useContext(I18nContext);
  if (!context) {
    throw new Error('useI18n must be used within an I18nProvider');
  }
  return context;
};
```

### 5. New File: `src/services/PdfService.ts`
```typescript
export class PdfService {
  static async exportToPdf(elementId: string, filename: string = 'invoice.pdf'): Promise<void> {
    // Dynamic import to reduce bundle size
    const [{ default: jsPDF }, { default: html2canvas }] = await Promise.all([
      import('jspdf'),
      import('html2canvas')
    ]);

    const element = document.getElementById(elementId);
    if (!element) {
      throw new Error(`Element with id "${elementId}" not found`);
    }

    try {
      // Create canvas from HTML element
      const canvas = await html2canvas(element, {
        scale: 2,
        useCORS: true,
        allowTaint: true,
        backgroundColor: '#ffffff'
      });

      const imgData = canvas.toDataURL('image/png');
      const pdf = new jsPDF();
      
      const imgWidth = 210; // A4 width in mm
      const pageHeight = 295; // A4 height in mm
      const imgHeight = (canvas.height * imgWidth) / canvas.width;
      let heightLeft = imgHeight;

      let position = 0;

      // Add first page
      pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
      heightLeft -= pageHeight;

      // Add additional pages if needed
      while (heightLeft >= 0) {
        position = heightLeft - imgHeight;
        pdf.addPage();
        pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
        heightLeft -= pageHeight;
      }

      pdf.save(filename);
    } catch (error) {
      console.error('Error generating PDF:', error);
      throw new Error('Failed to generate PDF');
    }
  }

  static async generateInvoicePdf(invoiceData: any): Promise<void> {
    const filename = `invoice-${invoiceData.number || Date.now()}.pdf`;
    await this.exportToPdf('tableeditor', filename);
  }
}
```

### 6. New File: `src/components/PDFExport/PDFExport.tsx`
```typescript
import React, { useState } from 'react';
import { IonButton, IonIcon, IonLoading } from '@ionic/react';
import { downloadOutline } from 'ionicons/icons';
import { PdfService } from '../../services/PdfService';
import { useI18n } from '../../context/I18nContext';
import { useAuth } from '../../context/AuthContext';
import { Permission } from '../../types/auth';

interface PDFExportProps {
  elementId?: string;
  filename?: string;
  invoiceData?: any;
}

export const PDFExport: React.FC<PDFExportProps> = ({ 
  elementId = 'tableeditor', 
  filename,
  invoiceData 
}) => {
  const [isExporting, setIsExporting] = useState(false);
  const { t } = useI18n();
  const { hasPermission } = useAuth();

  const handleExport = async () => {
    if (!hasPermission(Permission.EXPORT_PDF)) {
      alert(t('errors.noPermission'));
      return;
    }

    setIsExporting(true);
    
    try {
      if (invoiceData) {
        await PdfService.generateInvoicePdf(invoiceData);
      } else {
        const pdfFilename = filename || `export-${Date.now()}.pdf`;
        await PdfService.exportToPdf(elementId, pdfFilename);
      }
    } catch (error) {
      console.error('Export failed:', error);
      alert(t('errors.exportFailed'));
    } finally {
      setIsExporting(false);
    }
  };

  return (
    <>
      <IonButton 
        onClick={handleExport} 
        disabled={isExporting}
        fill="outline"
        size="default"
      >
        <IonIcon icon={downloadOutline} slot="start" />
        {t('buttons.exportPdf')}
      </IonButton>
      
      <IonLoading
        isOpen={isExporting}
        message={t('messages.generatingPdf')}
      />
    </>
  );
};
```

### 7. New File: `src/components/ThemeToggle/ThemeToggle.tsx`
```typescript
import React from 'react';
import { IonButton, IonIcon } from '@ionic/react';
import { moonOutline, sunnyOutline } from 'ionicons/icons';
import { useTheme } from '../../context/ThemeContext';

export const ThemeToggle: React.FC = () => {
  const { isDarkMode, toggleDarkMode } = useTheme();

  return (
    <IonButton 
      fill="clear" 
      onClick={toggleDarkMode}
      aria-label={isDarkMode ? 'Switch to light mode' : 'Switch to dark mode'}
    >
      <IonIcon icon={isDarkMode ? sunnyOutline : moonOutline} />
    </IonButton>
  );
};
```

### 8. New File: `src/components/LanguageSelector/LanguageSelector.tsx`
```typescript
import React from 'react';
import { IonSelect, IonSelectOption, IonItem, IonLabel } from '@ionic/react';
import { useI18n, Language } from '../../context/I18nContext';

export const LanguageSelector: React.FC = () => {
  const { language, setLanguage, t } = useI18n();

  const handleLanguageChange = (event: CustomEvent) => {
    setLanguage(event.detail.value as Language);
  };

  return (
    <IonItem>
      <IonLabel>{t('settings.language')}</IonLabel>
      <IonSelect
        value={language}
        onIonChange={handleLanguageChange}
        interface="popover"
      >
        <IonSelectOption value="en">English</IonSelectOption>
        <IonSelectOption value="es">Español</IonSelectOption>
      </IonSelect>
    </IonItem>
  );
};
```

### 9. New File: `src/components/UserRoleGuard/UserRoleGuard.tsx`
```typescript
import React, { ReactNode } from 'react';
import { useAuth } from '../../context/AuthContext';
import { UserRole, Permission } from '../../types/auth';

interface UserRoleGuardProps {
  children: ReactNode;
  roles?: UserRole | UserRole[];
  permissions?: Permission | Permission[];
  fallback?: ReactNode;
}

export const UserRoleGuard: React.FC<UserRoleGuardProps> = ({ 
  children, 
  roles,
  permissions,
  fallback = null 
}) => {
  const { hasRole, hasPermission } = useAuth();

  const hasRequiredRole = roles ? hasRole(roles) : true;
  const hasRequiredPermissions = permissions 
    ? (Array.isArray(permissions) 
        ? permissions.every(p => hasPermission(p))
        : hasPermission(permissions))
    : true;

  if (hasRequiredRole && hasRequiredPermissions) {
    return <>{children}</>;
  }

  return <>{fallback}</>;
};
```

### 10. New File: `src/translations/en.json`
```json
{
  "common": {
    "loading": "Loading...",
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "create": "Create",
    "search": "Search"
  },
  "buttons": {
    "exportPdf": "Export PDF",
    "login": "Login",
    "logout": "Logout"
  },
  "messages": {
    "generatingPdf": "Generating PDF...",
    "loginSuccess": "Login successful",
    "logoutSuccess": "Logged out successfully"
  },
  "errors": {
    "noPermission": "You don't have permission to perform this action",
    "exportFailed": "Failed to export PDF",
    "loginFailed": "Login failed"
  },
  "settings": {
    "language": "Language",
    "theme": "Theme",
    "darkMode": "Dark Mode"
  },
  "invoice": {
    "title": "Invoice",
    "number": "Invoice Number",
    "date": "Invoice Date",
    "billTo": "Bill To",
    "from": "From",
    "description": "Description",
    "amount": "Amount",
    "total": "Total"
  }
}
```

### 11. New File: `src/translations/es.json`
```json
{
  "common": {
    "loading": "Cargando...",
    "save": "Guardar",
    "cancel": "Cancelar",
    "delete": "Eliminar",
    "edit": "Editar",
    "create": "Crear",
    "search": "Buscar"
  },
  "buttons": {
    "exportPdf": "Exportar PDF",
    "login": "Iniciar Sesión",
    "logout": "Cerrar Sesión"
  },
  "messages": {
    "generatingPdf": "Generando PDF...",
    "loginSuccess": "Inicio de sesión exitoso",
    "logoutSuccess": "Sesión cerrada exitosamente"
  },
  "errors": {
    "noPermission": "No tienes permiso para realizar esta acción",
    "exportFailed": "Error al exportar PDF",
    "loginFailed": "Error en el inicio de sesión"
  },
  "settings": {
    "language": "Idioma",
    "theme": "Tema",
    "darkMode": "Modo Oscuro"
  },
  "invoice": {
    "title": "Factura",
    "number": "Número de Factura",
    "date": "Fecha de Factura",
    "billTo": "Facturar A",
    "from": "De",
    "description": "Descripción",
    "amount": "Cantidad",
    "total": "Total"
  }
}
```

### 12. Modified File: `src/App/App.tsx` (converted from App.js)
```typescript
import React, { Component } from "react";
import "./App.css";
import * as AppGeneral from "../socialcalc/AppGeneral";
import { DATA } from "../app-data.js";

import Menu from "../Menu/Menu";
import Files from "../Files/Files";
import { ConnectKitButton } from "connectkit";
import { PDFExport } from "../components/PDFExport/PDFExport";
import { ThemeToggle } from "../components/ThemeToggle/ThemeToggle";
import { LanguageSelector } from "../components/LanguageSelector/LanguageSelector";
import { UserRoleGuard } from "../components/UserRoleGuard/UserRoleGuard";
import { useAuth } from "../context/AuthContext";
import { useI18n } from "../context/I18nContext";
import { Permission, UserRole } from "../types/auth";

interface AppState {
  selectedFile: string;
  device: string;
  listFiles: boolean;
}

class App extends Component<{}, AppState> {
  constructor(props: {}) {
    super(props);
    this.state = {
      selectedFile: "default",
      device: AppGeneral.getDeviceType(),
      listFiles: false,
    };
    this.updateSelectedFile = this.updateSelectedFile.bind(this);
    this.toggleListFiles = this.toggleListFiles.bind(this);
  }

  updateSelectedFile(selectedFile: string) {
    this.setState({
      selectedFile: selectedFile,
    });
  }

  toggleListFiles() {
    this.setState((prevState) => ({
      listFiles: !prevState.listFiles,
    }));
  }

  componentDidMount() {
    let data = DATA[this.state.device];
    AppGeneral.initializeApp(data);
  }

  componentDidUpdate(prevProps: {}, prevState: AppState) {
    if (prevState.selectedFile !== this.state.selectedFile) {
      let data = DATA[this.state.device];
      AppGeneral.initializeApp(data);
    }
  }

  render() {
    return (
      <div className="App">
        <div className="App-header">
          <div className="header-content">
            <h2>Invoice Suite</h2>
            <div className="header-controls">
              <UserRoleGuard permissions={Permission.EXPORT_PDF}>
                <PDFExport />
              </UserRoleGuard>
              <LanguageSelector />
              <ThemeToggle />
              <ConnectKitButton />
            </div>
          </div>
        </div>

        <div className="App-body">
          <UserRoleGuard roles={[UserRole.ADMIN, UserRole.MANAGER]}>
            <Menu
              device={this.state.device}
              selectedFile={this.state.selectedFile}
              updateSelectedFile={this.updateSelectedFile}
              listFiles={this.state.listFiles}
              toggleListFiles={this.toggleListFiles}
            />
          </UserRoleGuard>

          <div className="main-content">
            {this.state.listFiles && (
              <UserRoleGuard permissions={Permission.VIEW_ALL_INVOICES}>
                <Files
                  updateSelectedFile={this.updateSelectedFile}
                  device={this.state.device}
                  toggleListFiles={this.toggleListFiles}
                />
              </UserRoleGuard>
            )}
            
            <div 
              id="workbookControl" 
              style={{ 
                height: "auto", 
                width: "auto", 
                backgroundColor: "white" 
              }}
            >
              <div 
                id="tableeditor" 
                style={{ 
                  height: this.state.device === "iPad" ? "1200px" : "auto", 
                  width: "auto" 
                }}
              ></div>
            </div>
          </div>
        </div>
      </div>
    );
  }
}

export default App;
```

### 13. Modified File: `src/index.tsx` (converted from index.js)
```typescript
import React from "react";
import App from "./App/App";
import "./index.css";
import { createRoot } from "react-dom/client";
import { Web3Provider } from "./utils/Web3Provider";
import { AuthProvider } from "./context/AuthContext";
import { ThemeProvider } from "./context/ThemeContext";
import { I18nProvider } from "./context/I18nContext";

const domNode = document.getElementById("root")!;
const root = createRoot(domNode);

root.render(
  <React.StrictMode>
    <I18nProvider>
      <ThemeProvider>
        <AuthProvider>
          <Web3Provider>
            <App />
          </Web3Provider>
        </AuthProvider>
      </ThemeProvider>
    </I18nProvider>
  </React.StrictMode>
);
```

### 14. Modified File: `package.json`
```json
{
  "name": "react-webapp",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@tanstack/react-query": "^5.49.2",
    "@testing-library/jest-dom": "^5.17.0",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "@web3-storage/w3up-client": "^17.2.0",
    "@ionic/react": "^7.0.0",
    "@ionic/core": "^7.0.0",
    "connectkit": "^1.8.2",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-scripts": "5.0.1",
    "viem": "2.x",
    "wagmi": "^2.10.9",
    "web-vitals": "^2.1.4",
    "whatwg-fetch": "^2.0.3",
    "jspdf": "^2.5.1",
    "html2canvas": "^1.4.1",
    "ionicons": "^7.0.0",
    "typescript": "^4.9.5",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@types/node": "^16.18.0"
  },
  "devDependencies": {
    "@babel/plugin-proposal-private-property-in-object": "^7.21.11",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

### 15. New File: `src/App/App.css` (additional styles)
```css
/* Dark mode styles */
body.dark {
  background-color: #1a1a1a;
  color: #ffffff;
}

body.dark .App-header {
  background-color: #2a2a2a;
  border-bottom: 1px solid #444;
}

body.dark #workbookControl {
  background-color: #2a2a2a !important;
}

body.dark #tableeditor {
  background-color: #2a2a2a;
  color: #ffffff;
}

/* Header styles */
.App-header {
  padding: 1rem;
  border-bottom: 1px solid #ddd;
  background-color: #f8f9fa;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
}

.header-controls {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.App-body {
  display: flex;
  min-height: calc(100vh - 80px);
}

.main-content {
  flex: 1;
  padding: 1rem;
}

/* Mobile responsive */
@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    gap: 1rem;
  }
  
  .header-controls {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .App-body {
    flex-direction: column;
  }
}
```

## INSTRUCTIONS

1. **Install Dependencies**:
   ```bash
   npm install @ionic/react @ionic/core jspdf html2canvas ionicons typescript @types/react @types/react-dom @types/node
   ```

2. **TypeScript Configuration**:
   Create `tsconfig.json` in the project root:
   ```json
   {
     "compilerOptions": {
       "target": "es5",
       "lib": [
         "dom",
         "dom.iterable",
         "es6"
       ],
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
     "include": [
       "src"
     ]
   }
   ```

3. **Ionic CSS Import**:
   Add to `src/index.css`:
   ```css
   @import '~@ionic/react/css/core.css';
   @import '~@ionic/react/css/normalize.css';
   @import '~@ionic/react/css/structure.css';
   @import '~@ionic/react/css/typography.css';
   @import '~@ionic/react/css/padding.css';
   @import '~@ionic/react/css/float-elements.css';
   @import '~@ionic/react/css/text-alignment.css';
   @import '~@ionic/react/css/text-transformation.css';
   @import '~@ionic/react/css/flex-utils.css';
   @import '~@ionic/react/css/display.css';
   ```

4. **Environment Setup**:
   Create `.env` file:
   ```
   REACT_APP_API_URL=http://localhost:3001
   REACT_APP_VERSION=1.0.0
   ```

5. **Testing**:
   The features include:
   - **Role-based access control** with Admin, Manager, User, and Viewer roles
   - **PDF export functionality** using jsPDF and html2canvas
   - **Multi-language support** (English/Spanish
```
