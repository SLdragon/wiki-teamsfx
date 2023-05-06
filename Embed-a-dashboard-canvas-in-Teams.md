> We appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

The dashboard tab template from Teams Toolkit enables you to quickly get started with embedding a canvas containing multiple cards that provide an overview of data or content in Microsoft Teams while worrying less about dashboard layout.

## In this tutorial, you will learn

Basic Concepts:
 * [What are Teams tabs](https://learn.microsoft.com/microsoftteams/platform/tabs/what-are-tabs)
 * [App design guidelines for Tab](https://learn.microsoft.com/microsoftteams/platform/tabs/design/tabs)
 * [Fluent UI Library](https://react.fluentui.dev/?path=/docs/concepts-introduction--page) and [Fluent UI React Charting Examples](https://fluentuipr.z22.web.core.windows.net/heads/master/react-charting/demo/index.html#/)

Get started with Teams Toolkit:
 * [How to create a new dashboard tab](#create-a-new-dashboard-project)
 * [How to understand the dashboard tab project](#take-a-tour-of-your-app-source-code)
 * [How to understand the widget abstraction](#widget-abstraction)
 * [How to understand the dashboard abstraction](#dashboard-abstraction)

Customize the scaffolded app template:
 * [How to add a widget](#how-to-add-a-new-widget)
 * [How to add a dashboard](#how-to-add-a-new-dashboard)
 * [How to customize the widget](#customize-the-widget)
 * [How to include a data loader](#how-to-include-a-data-loader)
 * [How to handle empty state](#how-to-handle-empty-state)
 * [How to refresh data as scheduled](#how-to-refresh-data-as-scheduled)
 * [How to use Microsoft Graph Toolkit as widget content](#how-to-use-microsoft-graph-toolkit-as-widget-content)
 * [How to embed Power BI to Dashboard](#how-to-embed-power-bi-to-dashboard)
 * [How to customize the dashboard layout](#customize-the-dashboard-layout)
 * [How to add a Graph API call](#how-to-add-a-new-graph-api-call)

## Create a new dashboard project

### In Visual Studio Code

1. From Teams Toolkit side bar click `Create a new app` or select `Teams: Create a new app` from the command palette.

![image](https://user-images.githubusercontent.com/107838226/220565583-7e6145f1-6f07-4072-b343-b9f76cc63162.png)

2. Select `Tab`.

![image](https://user-images.githubusercontent.com/107838226/236426750-fe879ebe-977d-4156-856f-847519fbd4e8.png)

3. Select `Dashboard` from App Features Using a Tab section.

![image](https://user-images.githubusercontent.com/107838226/236426985-7294c44c-6484-43e3-9038-5b1abfda189d.png)

4. Select programming language.

![image](https://user-images.githubusercontent.com/107838226/236427166-e2b40f16-cbd4-4bca-b8b7-3c41809aaf1e.png)

5. Select a workspace folder.

![image](https://user-images.githubusercontent.com/107838226/236427291-2eb06a0e-5f6a-4dd8-a387-e89d591eabb5.png)

6. Enter an application name and then press enter.

![5](https://user-images.githubusercontent.com/107838226/236427374-8be8a186-9563-46e8-8a63-617341c3c92a.png)

### In TeamsFx CLI

* If you prefer interactive mode, execute `teamsfx new` command, then use the keyboard to go through the same flow as in Visual Studio Code.

* If you prefer non-interactive mode, enter all required parameters in one command. 

`teamsfx new --interactive false --capabilities "dashboard" --programming-language "typescript" --folder "./" --app-name dashboard-cli-001`

After you successfully created the project, you can quickly start local debugging via `F5` in VSCode. Select `Debug (Edge)` or `Debug (Chrome)` debug option of your preferred browser. after running this template and you will see a tab app loaded as below:

![dashboard](https://user-images.githubusercontent.com/107838226/236610252-743b8e89-e562-4d63-8bb8-a9195502d86f.png)

This app also supported teams different themes, including dark theme and high contrast theme.

|           Dark theme           |     High contrast theme      |
| :----------------------------: | :--------------------------: |
| <img src="https://user-images.githubusercontent.com/107838226/236610265-f7b7fb9e-eeac-4761-af11-c1a8da519519.png" /> | <img src="https://user-images.githubusercontent.com/107838226/236610276-36292c3b-b8a6-4f5d-a245-2940964ceabd.png" /> |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Take a tour of your app source code

This section walks through the generated code. The project folder contains the following:

| Folder       | Contents                                             |
|--------------|------------------------------------------------------|
| `.vscode`    | VSCode files for debugging.                          |
| `appPackage` | Templates for the Teams application manifest.        |
| `env`        | Environment files.                                   |
| `infra`      | Templates for provisioning Azure resources.          |
| `src`        | The source code for the dashboard Teams application. |

The following files provide the business logic for the dashboard tab. These files can be updated to fit your business logic requirements. The default implementation provides a starting point to help you get started.

| File                                 | Contents                                            |
| ------------------------------------ | --------------------------------------------------- |
| `src/models/chartModel.ts`           | Data model for the chart widget.                    |
| `src/models/listModel.ts`            | Data model for the list widget.                     |
| `src/services/chartService.ts`       | A data retrive implementation for the chart widget. |
| `src/services/listService.ts`        | A data retrive implementation for the list widget.  |
| `src/dashboards/SampleDashboard.tsx` | A sample dashboard layout implementation.           |
| `src/styles/ChartWidget.css`         | The chart widget style file.                        |
| `src/styles/ListWidget.css`          | The list widget style file.                         |
| `src/widgets/ChartWidget.tsx`        | A widget implementation that can display a chart.   |
| `src/widgets/ListWidget.tsx`         | A widget implementation that can display a list.    |
| `src/App.css`                        | The style of application route.                     |
| `src/App.tsx`                        | Application route.                                  |

The following files are project-related files. You generally will not need to customize these files.

| File                               | Contents                                                      |
| ---------------------------------- | ------------------------------------------------------------- |
| `src/index.css`                    | The style of application entry point.                         |
| `src/index.tsx`                    | Application entry point.                                      |
| `src/internal/addNewScopes.ts`     | Implementation of new scopes add.                             |
| `src/internal/context.ts`          | TeamsFx Context.                                              |
| `src/internal/login.ts`            | Implementation of login.                                      |
| `src/internal/singletonContext.ts` | Implementation of the TeamsUserCredential instance singleton. |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Widget abstraction

To simplify the development of a widget, the TeamsFx SDK provides a `BaseWidget` class 
for developers to inherit to quickly implement a widget that meets their needs without pay too much attention to how to implement the widget layout.

Below is the definition of the BaseWidget class.

```ts
export interface IWidgetClassNames {
  root?: string;
  header?: string;
  body?: string;
  footer?: string;
}

const classNames: IWidgetClassNames = mergeStyleSets({
  root: {
    display: "grid",
    padding: "1.25rem 2rem 1.25rem 2rem",
    backgroundColor: tokens.colorNeutralBackground1,
    border: "1px solid var(--colorTransparentStroke)",
    boxShadow: tokens.shadow4,
    borderRadius: tokens.borderRadiusMedium,
    gap: tokens.spacingHorizontalL,
    gridTemplateRows: "max-content 1fr max-content",
  },
  header: {
    display: "grid",
    height: "max-content",
    "& div": {
      display: "grid",
      gap: tokens.spacingHorizontalS,
      alignItems: "center",
      gridTemplateColumns: "min-content 1fr min-content",
    },
    "& svg": {
      height: "1.5rem",
      width: "1.5rem",
    },
    "& span": {
      fontWeight: tokens.fontWeightSemibold,
      lineHeight: tokens.lineHeightBase200,
      fontSize: tokens.fontSizeBase200,
    },
  },
  footer: {
    "& button": {
      width: "fit-content",
    },
  },
});

interface BaseWidgetState {
  loading?: boolean;
}

export class BaseWidget<P, S> extends Component<P, S & BaseWidgetState> {
  public constructor(props: Readonly<P>) {
    super(props);
    this.state = { loading: undefined } as S & BaseWidgetState;
  }

  public async componentDidMount() {
    this.setState({ ...(await this.getData()), loading: false });
  }

  public render() {
    const { root, header, body, footer } = this.styling();
    const showLoading = this.state.loading !== false && this.loading() !== undefined;
    return (
      <div className={mergeStyles(classNames.root, root)}>
        {this.header() && (
          <div className={mergeStyles(classNames.header, header)}>{this.header()}</div>
        )}
        {showLoading ? (
          this.loading()
        ) : (
          <>
            {this.body() !== undefined && <div className={body}>{this.body()}</div>}
            {this.footer() !== undefined && (
              <div className={mergeStyles(classNames.footer, footer)}>{this.footer()}</div>
            )}
          </>
        )}
      </div>
    );
  }

  protected async getData(): Promise<S> {
    return undefined;
  }

  protected header(): JSX.Element | undefined {
    return undefined;
  }

  protected body(): JSX.Element | undefined {
    return undefined;
  }

  protected footer(): JSX.Element | undefined {
    return undefined;
  }

  protected loading(): JSX.Element | undefined {
    return undefined;
  }

  protected styling(): IWidgetClassNames {
    return {};
  }
}
```

| Methods               | Function                                                                                                                                                                             | Recommend to override |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| `constructor()`       | Assigns the initial `this.state` and call the constructor of the super class `React.Component`.                                                                                     | NO                    |
| `componentDidMount()` | This method will be invoked after a component is mounted and assigns a value to the `data` property of the state by calling the `getData()` method.                                 | NO                    |
| `render()`            | This method will be called each time an update happens, the dashboard default layout is defined in this method.                                                                      | NO                    |
| `getData()`           | This method can be used to get the data needed by the widget, and the value returned by this method will be set to `this.state.data`.                                                | YES                   |
| `header()`     | This method is used to define what the widget header will look like. You can choose to override this method to define the header of your widget, if not, the widget will not have a header. | YES                   |
| `body()`       | This method is used to define what the widget body will look like. You can choose to override this method to define the body of your widget, if not, the widget will not have a body.     | YES                   |
| `footer()`     | This method is used to define what the widget footer will look like. You can choose to override this method to define the footer of your widget, if not, the widget will not have a footer. | YES                   |
| `loading()`     | This method is typically called when the widget is in the process of fetching data. If a loading indicator is required, the method can return a `JSX.Element` containing the necessary components to render the loading indicator. | YES                   |
| `style()`     | Override this method to returns an object that defines the class names for the different parts of the widget. | YES                   |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Dashboard Abstraction
To facilitate the definition and customization of dashboard layout, TeamsFx offers a `BaseDashboard` class that developers can inherit to efficiently create their own dashboard.

The following is the definition of the `BaseDashboard` class.

```ts
function dashboardStyle(isMobile?: boolean) {
  return mergeStyles({
    display: "grid",
    gap: "20px",
    padding: "20px",
    gridTemplateRows: "1fr",
    gridTemplateColumns: "4fr 6fr",
    ...(isMobile === true ? { gridTemplateColumns: "1fr", gridTemplateRows: "1fr" } : {}),
  });
}

interface BaseDashboardState {
  isMobile?: boolean;
  showLogin?: boolean;
  observer?: ResizeObserver;
}

export class BaseDashboard<P, S> extends Component<P, S & BaseDashboardState> {
  private ref: React.RefObject<HTMLDivElement>;

  public constructor(props: Readonly<P>) {
    super(props);
    this.state = {
      isMobile: undefined,
      showLogin: undefined,
      observer: undefined,
    } as S & BaseDashboardState;
    this.ref = React.createRef<HTMLDivElement>();
  }

  public async componentDidMount() {
    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        if (entry.target === this.ref.current) {
          const { width } = entry.contentRect;
          this.setState({ isMobile: width < 600 } as S & BaseDashboardState);
        }
      }
    });
    observer.observe(this.ref.current!);
  }

  public componentWillUnmount(): void {
    if (this.state.observer && this.ref.current) {
      this.state.observer.unobserve(this.ref.current);
    }
  }

  public render() {
    return (
      <div
        ref={this.ref}
        className={mergeStyles(dashboardStyle(this.state.isMobile), this.styling())}
      >
        {this.layout()}
      </div>
    );
  }

  protected layout(): JSX.Element | undefined {
    return undefined;
  }

  protected styling(): string {
    return null;
  }
}
```

The TeamsFx SDK offers a set of basic layouts and customizable methods within the `BaseDashboard` class. Despite being a react component, the `BaseDashboard` class is equipped with pre-implemented functions that are based on the lifecycle of react components. This includes a basic render logic that utilizes the Grid layout, as well as an observer that automatically adjusts to mobile devices.


| Methods                  | Function                                                                                                       | Recommend to override |
|--------------------------|----------------------------------------------------------------------------------------------------------------|-----------------------|
| `constructor()`          | Initialize the state and variables of the dashboard.                                                           | NO                    |
| `componentDidMount()`    | This method will be called after a component has been mounted.                                                 | NO                    |
| `componentWillUnmount()` | This method will be called when a component is about to be unmounted.                                          | NO                    |
| `render()`               | This method will be called every time an update occurs, and it defines the default layout for the dashboard.   | NO                    |
| `layout()`               | To define the layout of the widget in the dashboard, you can override this method.                             | YES                   |
| `styling()`              | To customize the style of the dashboard, you can override this method.                                         | YES                   |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new widget

To add a new widget to the dashboard, you can follow these steps:

1. [Step 1: Define a data model](#step-1-define-a-data-model)
2. [Step 2: Create a data retrive service](#step-2-create-a-data-retrive-service)
3. [Step 3: Create a widget file](#step-3-create-a-widget-file)
4. [Step 4: Add the widget to the dashboard](#step-4-add-the-widget-to-the-dashboard)

### Step 1: Define a data model

To define a data model based on the business scenario, it is recommended to place the data model under the `src/models` directory. Here is an example of a data model:

```typescript
//sampleModel.ts
export interface SampleModel {
  content: string;
}
```

### Step 2: Create a data retrive service

Typically, a widget requires a service to retrieve the necessary data for displaying its content. This service can either fetch static data from a predefined source or retrieve dynamic data from a backend service or API.

For instance, we can implement a service that returns static data and place it under the `src/services` directory.

Here is a sample service for retrieving static data:

```typescript
//sampleService.ts
import { SampleModel } from "../models/sampleModel";

export const getSampleData = (): SampleModel => {
  return { content: "Hello world!" };
};
```

### Step 3: Create a widget file

Create a widget file in `src/widgets` folder. Inherit the `BaseWidget` class from `@microsoft/teamsfx-react`. The following table lists the methods that you can override to customize your widget.

| Methods     | Function                                                                                                                                      |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `getData()` | This method is utilized to retrieve data for the widget. It can be implemented to obtain data from either the backend service or the Microsoft Graph API. |
| `header()`  | Customize the header content of the widget.                                                                                                    |
| `body()`    | Customize the body content of the widget.                                                                                                      |
| `footer()`  | Customize the footer content of the widget.                                                                                                    |
| `styling()` | Customize the style of the widget.                                                                                                                    |

> All method overrides are optional.

Here's a sample widget implementation:

```tsx
//SampleWidget.tsx
import { Button, Text } from "@fluentui/react-components";
import { BaseWidget } from "@microsoft/teamsfx-react";
import { SampleModel } from "../models/sampleModel";
import { getSampleData } from "../services/sampleService";

interface SampleWidgetState {
  data?: SampleModel;
}

export class SampleWidget extends BaseWidget<any, SampleWidgetState> {
  override async getData(): Promise<SampleWidgetState> {
    return { data: getSampleData() };
  }

  override header(): JSX.Element | undefined {
    return <Text>Sample Widget</Text>;
  }

  override body(): JSX.Element | undefined {
    return <div>{this.state.data?.content}</div>;
  }

  override footer(): JSX.Element | undefined {
    return <Button>View Details</Button>;
  }
}
```

### Step 4: Add the widget to the dashboard

Please navigate to the `src/dashboards/SampleDashboard.tsx` file and include the widget in the implementation of the `layout()` method. In case you wish to create a new dashboard, kindly refer to [How to add a new dashboard](#how-to-add-a-new-dashboard).

```tsx
override layout(): JSX.Element | undefined {
  return (
    <>
      <ListWidget />
      <ChartWidget />
      <SampleWidget />
    </>
  );
}
```

Optional: If you prefer to arrange multiple widgets within the same column, you may refer to the following code snippet:

```css
.one-column {
  display: grid;
  gap: 20px;
  grid-template-rows: 1fr 1fr;
}
```

```jsx
override layout(): JSX.Element | undefined {
  return (
    <>
      <ListWidget />
      <div className="one-column">
        <ChartWidget />
        <SampleWidget />
      </div>
    </>
  );
}
```

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new dashboard

To add a new dashboard layout, you can follow these steps:

1. [Step 1: Create a dashboard class](#step-1-create-a-dashboard-class)
2. [Step 2: Override methods to customize dashboard layout](#step-2-override-methods-to-customize-dashboard-layout)
3. [Step 3: Add a route for the new dashboard](#step-3-add-a-route-for-the-new-dashboard)
4. [Step 4: Modify manifest to add a new dashboard tab](#step-4-modify-manifest-to-add-a-new-dashboard-tab)

### Step 1: Create a dashboard class

In the `src/dashboards` directory, generate a file with the extension `.tsx` and name it `YourDashboard.tsx`. Within this file, create a class that extends the `BaseDashboard` class from `@microsoft/teamsfx-react`.

```tsx
//YourDashboard.tsx
import { BaseDashboard } from "@microsoft/teamsfx-react";

export default class YourDashboard extends BaseDashboard<any, any> {}
```

### Step 2: Override methods to customize dashboard layout

You have the ability to customize the layout of the dashboard by overriding certain methods provided by the `BaseDashboard` class. The table below outlines the methods that can be overridden:

| Methods     | Function                             |
| ----------- | ------------------------------------ |
| `styling()` | Customize the style of the dashboard |
| `layout()`  | Define widgets layout                |

Here is an example of how to customize the layout of a dashboard.

```css
.your-dashboard-layout {
  grid-template-columns: 6fr 4fr;
}
```

```tsx
import { BaseDashboard } from "@microsoft/teamsfx-react";
import ListWidget from "../widgets/ListWidget";
import ChartWidget from "../widgets/ChartWidget";

export default class YourDashboard extends BaseDashboard<any, any> {
  override styling(): string {
    return "your-dashboard-layout";
  }

  override layout(): JSX.Element | undefined {
    return (
      <>
        <ListWidget />
        <ChartWidget />
      </>
    );
  }
}
```

### Step 3: Add a route for the new dashboard

Open the `src/App.tsx` file, and add a route for the new dashboard. Here is an example:

```tsx
import YourDashboard from "./dashboards/YourDashboard";

export default function App() {
  ...
  <Route path="/yourdashboard" element={<YourDashboard />} />
  ...
}
```

### Step 4: Modify manifest to add a new dashboard tab

Open the `appPackage/manifest.json` file and add a new dashboard tab under the `staticTabs`. Here is an example:

```json
{
  "entityId": "index1",
  "name": "Your Dashboard",
  "contentUrl": "${{TAB_ENDPOINT}}/index.html#/yourdashboard",
  "websiteUrl": "${{TAB_ENDPOINT}}/index.html#/yourdashboard",
  "scopes": ["personal"]
}
```

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Customize the Widget
As the `BaseWidget` class offers several methods that are open for customization, one may customize a widget by overriding these methods.

- Override `header()`, `body()` and `footer()` methods to customize the widget.

```tsx
export class NewsWidget extends BaseWidget<any, any> {
  override header(): JSX.Element | undefined {
    return (
      <div>
        <News28Regular />
        <Text>Your News</Text>
        <Button icon={<MoreHorizontal32Regular />} appearance="transparent" />
      </div>
    );
  }

  override body(): JSX.Element | undefined {
    return (
      <div>
        <Image src="image.svg" />
        <Text>Lorem Ipsum Dolor</Text>
        <Text>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Enim,
          elementum sed
        </Text>
      </div>
    );
  }

  override footer(): JSX.Element | undefined {
    return (
      <Button
        appearance="transparent"
        icon={<ArrowRight16Filled />}
        iconPosition="after"
        size="small"
      >
        View details
      </Button>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209327297-bbcc62fd-9ec1-4a4e-88af-f408085634aa.png)


- Override `body() and `footer()` methods to customize the widget.

```tsx
export class NewsWidget extends BaseWidget<any, any> {
  override body(): JSX.Element | undefined {
    return (
      <div>
        <Image src="image.svg" />
        <Text>Lorem Ipsum Dolor</Text>
        <Text>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Enim,
          elementum sed
        </Text>
      </div>
    );
  }

  override footer(): JSX.Element | undefined {
    return (
      <Button
        appearance="transparent"
        icon={<ArrowRight16Filled />}
        iconPosition="after"
        size="small"
      >
        View details
      </Button>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209326796-69fa92a6-0582-4ddb-bb1c-8383ac805433.png)


- Override the `body()` method to customize the widget.

```tsx
export class NewsWidget extends BaseWidget<any, any> {
  override body(): JSX.Element | undefined {
    return (
      <div>
        <Image src="image.svg" />
        <Text>Lorem Ipsum Dolor</Text>
        <Text>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Enim,
          elementum sed
        </Text>
      </div>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209326667-e88e000f-04c5-48e6-a976-b6e40237a9ad.png)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to include a data loader

To incorporate a data loader that runs before the widget is loaded, you can override the `loading()` method in your widget class. Here is an example:

```tsx
override loading(): JSX.Element | undefined {
  return (
    <div className="loading">
      <Spinner label="Loading..." labelPosition="below" />
    </div>
  );
}
```

Presently, a loading spinner is displayed during the data loading process. Once the data has been loaded, the loading spinner is concealed, and the list data and footer button are exhibited.

![5xcq3-utv28](https://user-images.githubusercontent.com/107838226/212793039-90aa6f7e-28fd-4650-bb64-f9c3859d260b.gif)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to handle empty state

To exhibit a particular content within your widget when the data is empty, you must adjust the `body()` method within your widget file to accommodate distinct data states. The ensuing example demonstrates how to present an empty image when the data within the **ListWidget** is absent:

```tsx
  override body(): JSX.Element | undefined {
    let hasData = this.state.data && this.state.data.length > 0;
    return (
      <div>
        {hasData ? (
          <>
            {this.state.data?.map((t: ListModel) => {
              ...
            })}
          </>
        ) : (
          <div>
            <Image src="empty-default.svg" height="150px" />
            <Text align="center">No data</Text>
          </div>
        )}
      </div>
    );
  }
```

Likewise, you can utilize a comparable technique to eliminate the footer content of your widget when the data is vacant.

```ts
  override footer(): JSX.Element | undefined {
    let hasData = this.state.data && this.state.data.length > 0;
    if (hasData) {
      return <Button>...</Button>;
    }
  }
```

When the data is vacant, your list widget will appear as follows:
![image](https://user-images.githubusercontent.com/107838226/212546123-33b8943e-094c-4de5-9f23-61868ee4e51b.png)


<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to refresh data as scheduled

The subsequent example delineates how to present live data within a widget. The widget showcases the current time and refreshes every second.

```tsx
interface IRefreshWidgetState {
  data: string;
}

export class RefreshWidget extends BaseWidget<any, IRefreshWidgetState> {
  override body(): JSX.Element | undefined {
    return <>{this.state.data}</>;
  }

  async componentDidMount() {
    setInterval(() => {
      this.setState({ data: new Date().toLocaleTimeString() });
    }, 1000);
  }
}
```

You may customize the `setInterval` method to invoke your own function for data refreshment, as demonstrated below: `setInterval(() => yourGetDataFunction(), 1000)`.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to use Microsoft Graph Toolkit as widget content

The Microsoft Graph Toolkit comprises a collection of reusable, framework-agnostic web components and utilities that facilitate accessing and manipulating Microsoft Graph. You can leverage the Microsoft Graph Toolkit with any web framework or even without a framework.

To utilize the Microsoft Graph Toolkit as your widget content, you can follow the steps outlined below:

### Step 1: Add SSO functionality to your Teams app

Microsoft Teams furnishes a single sign-on (SSO) feature for an application to acquire a signed-in Teams user token and access Microsoft Graph. For instructions on how to incorporate SSO functionality into your Teams app, please consult [this document](https://aka.ms/teamsfx-add-sso-new).

### Step 2: Install required npm packages

Execute the subsequent command in your project's root directory to install the necessary npm packages:

```bash
npm install @microsoft/mgt-react @microsoft/mgt-element @microsoft/mgt-teamsfx-provider
```

### Step 3: Add a new Graph Toolkit widget

Generate a new widget file in the `src/views/widgets` directory of your project, for instance, `GraphWidget.tsx`. Within this widget, we will assist users in granting consent to our application to access Microsoft Graph, and subsequently display the user's to-do list by leveraging the Microsoft Graph Toolkit. The code snippet below is an exemplar of how to use the Todo component from the Microsoft Graph Toolkit within the widget.

```tsx
import { Providers, ProviderState, Todo } from "@microsoft/mgt-react";
import { TeamsFxProvider } from "@microsoft/mgt-teamsfx-provider";

import { loginAction } from "../../internal/login";
import { TeamsUserCredentialContext } from "../../internal/singletonContext";
import { BaseWidget } from "@microsoft/teamsfx-react";

interface IGraphWidgetState {
  needLogin: boolean;
}

export class GraphWidget extends Widget<any, IGraphWidgetState> {
  override body(): JSX.Element | undefined {
    return <div>{this.state.needLogin === false && <Todo />}</div>;
  }

  async componentDidMount() {
    super.componentDidMount();

    // Initialize TeamsFx provider
    const provider = new TeamsFxProvider(TeamsUserCredentialContext.getInstance().getCredential(), [
      "Tasks.ReadWrite",
    ]);
    Providers.globalProvider = provider;

    // Check if user is signed in
    if (await this.checkIsConsentNeeded()) {
      await loginAction(["Tasks.ReadWrite"]);
    }

    // Update signed in state
    Providers.globalProvider.setState(ProviderState.SignedIn);
    this.setState({ needLogin: false });
  }

  /**
   * Check if user needs to consent
   * @returns true if user needs to consent
   */
  async checkIsConsentNeeded() {
    let needConsent = false;
    try {
      await TeamsUserCredentialContext.getInstance().getCredential().getToken(["Tasks.ReadWrite"]);
    } catch (error) {
      needConsent = true;
    }
    return needConsent;
  }
}
```

Additionally, alternative Microsoft Graph Toolkit components can also be utilized within your widget. For further details regarding Microsoft Graph Toolkit components, please consult [this document](https://learn.microsoft.com/en-us/graph/toolkit/overview).

### Step 4: Incorporate the widget into the dashboard layout

Integrate the newly created widget into your dashboard file.

```tsx
...
export default class YourDashboard extends BaseDashboard<any, any> {
  ...
  override layout(): undefined | JSX.Element {
    return (
      <>
        <GraphWiget />
      </>
    );
  }
  ...
}
```

Upon launching or refreshing your Teams application, subsequent to completing the login process and consenting to our application's access to Microsoft Graph, the newly created widget utilizing the Microsoft Graph Toolkit will be visible.

![image](https://user-images.githubusercontent.com/107838226/213122528-f0823063-b5b6-4cbf-9f2c-0198e6d72958.png)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to embed Power BI to Dashboard

For instructions on how to embed a Power BI item into the dashboard, please refer to [this document](https://learn.microsoft.com/en-us/javascript/api/overview/powerbi/powerbi-client-react).

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Customize the Dashboard Layout

The TeamsFx offers several convenient methods for defining and modifying the dashboard layout.

- Arrange three widgets horizontally with a height of 350px, utilizing 20%, 60%, and 20% of the width correspondingly.

```css
.customize-class-name {
  grid-template-rows: 350px;
  grid-template-columns: 2fr 6fr 2fr;
}
```

```tsx
export default class SampleDashboard extends BaseDashboard<any, any> {

  override styling(): string {
    return "customize-class-name";
  }

  override layout(): JSX.Element | undefined {
    return (
      <>
        <ListWidget />
        <ChartWidget />
        <NewsWidget />
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209525799-9ee5b7ab-6263-4a1f-89e6-b33eee6d31dc.png)

- Place two widgets side by side, with widths of 600px and 1100px respectively. The height of the first widget will be determined by the maximum height of its content, while the height of the second widget will be fixed at 400px.

```css
.customize-class-name {
  grid-template-rows: max-content 400px;
  grid-template-columns: 600px 1100px;
}
```

```tsx
export default class SampleDashboard extends Dashboard {
  override styling(): string {
    return "customize-class-name";
  }

  override layout(): JSX.Element | undefined {
    return (
      <>
        <ListWidget />
        <ChartWidget />
        <NewsWidget />
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209532741-0ad90649-f904-4dae-af25-940e7f874bf2.png)

- Organize two widgets vertically in a column.

```css
.one-column {
  display: grid;
  gap: 20px;
  grid-template-rows: 1fr 1fr;
}
```

```tsx
export default class SampleDashboard extends BaseDashboard<any, any> {
  override layout(): JSX.Element | undefined {
    return (
      <>
        <NewsWidget />
        <div className="one-column">
          <ListWidget />
          <ChartWidget />          
        </div>
      </>
    );
  }
}
```

![image](https://user-images.githubusercontent.com/107838226/209540272-094c871c-cdf0-45a4-b131-b1548e182e6f.png)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new Graph API call

### Add SSO First

Prior to implementing your Graph API call logic, it is necessary to enable Single Sign-On (SSO) for your dashboard project. For instructions on how to add SSO to your project, please refer to [this document](https://aka.ms/teamsfx-add-sso-new);

Assuming that you have successfully added the SSO files to your project, you are now able to invoke Graph APIs. It is important to note that there are two categories of Graph APIs: those that are called from the front-end (which typically require delegated permissions), and those that are called from the back-end (such as sendActivityNotification, which typically require application permissions). To determine the type of permissions required for the Graph APIs you intend to call, please refer to [this tutorial](https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-beta).

### Call Graph API from the front-end (use delegated permissions)

To invoke a Graph API from the front-end tab, you may follow the steps outlined below.

1. [Step 1: Consent delegated permissions first] (#step-1-consent-delegated-permissions-first)
2. [Step 2: Create a graph client by adding the scope related to the Graph API you want to call] (#step-2-create-a-graph-client-by-adding-the-scope-related-to-the-graph-api-you-want-to-call)
3. [Step 3: Call the Graph API, and parse the response into a certain model, which will be used by front-end] (#step-3-call-the-graph-api-and-parse-the-response-into-a-certain-model-which-will-be-used-by-front-end)

#### Step 1: Consent delegated permissions first

To obtain the name of the permission scope associated with the Graph API you intend to invoke, please refer to [the Graph API V1.0] (https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-1.0).

#### Step 2: Create a graph client by adding the scope related to the Graph API you want to call

You can refer to the following code snippet:

```ts
let credential: TeamsUserCredential;  
credential = TeamsUserCredentialContext.getInstance().getCredential();
const graphClient: Client = createMicrosoftGraphClientWithCredential(credential, scope);
```

#### Step 3: Call the Graph API, and parse the response into a certain model, which will be used by front-end

You can refer to the following code snippet:

```ts
try {
  const graphApiResult = await graphClient.api("<GRAPH_API_PATH>").get();
  // Parse the graphApiResult into a Model you defined, used by the front-end.
} catch (e) {}
```

### Call graph api from the back-end(use application permissions)

If you want to call a Graph API from the back-end, you can refer to the following steps.

1. [Step 1: Consent application permissions first](#step-1-consent-application-permissions-first)
2. [Step 2: Add an Azure Function](#step-2-add-an-azure-function)
3. [Step 3: Add your logic in Azure Function](#step-4-add-your-logic-in-azure-function)
4. [Step 4: Call the Azure Function from the front-end](#step-5-call-the-azure-function-from-the-front-end)

#### Step 1: Consent application permissions first

Go to [Azure portal](https://portal.azure.com/) > Click `Azure Active Directory` > Click `App registrations` in the side bar > Click your Dashboard app > Click `API permissions` in the side bar > Click `+Add a permission` > Choose `Microsoft Graph` > Choose `Application permissions` > Find the permissions you need > Click `Add permissions` button in the bottom > Click `✔Grant admin consent for XXX` and then click `Yes` button to finish the admin consent

#### Step 2: Add an Azure Function

For how to add an Azure Function to your project you can refer to [this doc](https://aka.ms/teamsfx-add-azure-function).

#### Step 3: Add your logic in Azure Function

In the `index.jsx`/`index.ts` under the folder named in step 2, you can add your logic which contains back-end Graph API calling with application permissions. You can refer to the following code snippet.

```ts
/**
 * This function handles requests from teamsfx client.
 * The HTTP request should contain an SSO token queried from Teams in the header.
 * Before trigger this function, teamsfx binding would process the SSO token and generate teamsfx configuration.
 *
 * You should initializes the teamsfx SDK with the configuration and calls these APIs.
 *
 * The response contains multiple message blocks constructed into a JSON object, including:
 * - An echo of the request body.
 * - The display name encoded in the SSO token.
 * - Current user's Microsoft 365 profile if the user has consented.
 *
 * @param {Context} context - The Azure Functions context object.
 * @param {HttpRequest} req - The HTTP request.
 * @param {teamsfxContext} TeamsfxContext - The context generated by teamsfx binding.
 */
export default async function run(
  context: Context,
  req: HttpRequest,
  teamsfxContext: TeamsfxContext
): Promise<Response> {
  context.log("HTTP trigger function processed a request.");

  // Initialize response.
  const res: Response = {
    status: 200,
    body: {},
  };

  // Your logic here.

  return res;
}
```
#### Step 4: Call the Azure Function from the front-end

Call the Azure Function by function name. You can refer to the following code snippet to call the Azure Function.

```ts
const functionName = process.env.REACT_APP_FUNC_NAME || "myFunc";
export let taskName: string;

export async function callFunction(params?: string) {
  taskName = params || "";
  const credential = TeamsUserCredentialContext.getInstance().getCredential();
  if (!credential) {
    throw new Error("TeamsFx SDK is not initialized.");
  }
  try {
    const apiBaseUrl = process.env.REACT_APP_FUNC_ENDPOINT + "/api/";    
    const apiClient = createApiClient(
      apiBaseUrl,
      new BearerTokenAuthProvider(async () => (await credential.getToken(""))!.token)
    );
    const response = await apiClient.get(functionName);
    return response.data;
  } catch (err: unknown) {
    ...
  }
}
```

Refer to [this sample](https://github.com/OfficeDev/TeamsFx-Samples/blob/dev/hello-world-tab-with-backend/tabs/src/components/sample/AzureFunctions.tsx) for some helps. And you can read [this doc](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference?tabs=blob) for more details.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>
