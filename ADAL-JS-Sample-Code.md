```javascript
import { Version } from '@microsoft/sp-core-library';
import {
  BaseClientSideWebPart,
  IPropertyPaneConfiguration,
  PropertyPaneTextField
} from '@microsoft/sp-webpart-base';
import { escape } from '@microsoft/sp-lodash-subset';

import styles from './HelloWorldWebPart.module.scss';
import * as strings from 'HelloWorldWebPartStrings';

import * as AuthenticationContext from 'adal-angular';

import '../WebPartAuthenticationContext';

export interface IHelloWorldWebPartProps {
  description: string;
}

export default class HelloWorldWebPart extends BaseClientSideWebPart<IHelloWorldWebPartProps> {

  public render(): void {
    this.domElement.innerHTML = `
      <div class="${ styles.helloWorld }">
        <div class="${ styles.container }">
          <div class="${ styles.row }">
            <div class="${ styles.column }">
              <span class="${ styles.title }">Welcome to SharePoint!</span>
              <p class="${ styles.subTitle }">Customize SharePoint experiences using Web Parts.</p>
              <p class="${ styles.description }">${escape(this.properties.description)}</p>
              <a href="https://aka.ms/spfx" class="${ styles.button }">
                <span class="${ styles.label }">Learn more</span>
              </a>
            </div>
          </div>
        </div>
      </div>`;

      var authContext = new AuthenticationContext({
        clientId: 'your application ID',
        instance: "https://login.microsoftonline.com/",
        tenant: "tenant-account.onmicrosoft.com",
        // postLogoutRedirectUri: 'https://localhost:4321/temp/workbench.html'
      });
      // Make an AJAX request to the Microsoft Graph API and print the response as JSON.
      var getToken;
      var getCurrentUser = function (access_token) {
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'https://graph.microsoft.com/v1.0/me', true);
        xhr.setRequestHeader('Authorization', 'Bearer ' + access_token);
        xhr.onreadystatechange = function () {
          if (xhr.readyState === 4 && xhr.status === 200) {
            // Do something with the response
            
            getToken=JSON.stringify(JSON.parse(xhr.responseText), null, '  ');
            console.log('get Graph APi information=='+getToken);
          } else {
            // TODO: Do something with the error (or non-200 responses)
          //  console.log(' error');
          }
        };
        xhr.send();
      }

      if (authContext.isCallback(window.location.hash)) {
        // Handle redirect after token requests
        authContext.handleWindowCallback();
        var err = authContext.getLoginError();
        if (err) {
          // TODO: Handle errors signing in and getting tokens
          console.log('login error');
        }
      } else {
        // If logged in, get access token and make an API request
        var user = authContext.getCachedUser();
        if (user) {
         

          // Get an access token to the Microsoft Graph API
          authContext.acquireToken(
            'https://graph.microsoft.com',
            function (error, token) {
              if (error || !token) {
                // TODO: Handle error obtaining access token
                console.log('Token error');
                return;
              }
              // Use the access token
              console.log('token=='+token);
              getCurrentUser(token);
            }
          );
        } else {
          authContext.login();
        }
       
      }
  }

  protected get dataVersion(): Version {
    return Version.parse('1.0');
  }

  protected getPropertyPaneConfiguration(): IPropertyPaneConfiguration {
    return {
      pages: [
        {
          header: {
            description: strings.PropertyPaneDescription
          },
          groups: [
            {
              groupName: strings.BasicGroupName,
              groupFields: [
                PropertyPaneTextField('description', {
                  label: strings.DescriptionFieldLabel
                })
              ]
            }
          ]
        }
      ]
    };
  }
}
```