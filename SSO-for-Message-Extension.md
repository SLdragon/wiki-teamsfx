# About Message Extension
Message extensions allow the users to interact with your web service through buttons and forms in the Microsoft Teams client. They can search or initiate actions in an external system from the compose message area, the command box, or directly from a message. You can send back the results of that interaction to the Teams client in the form of a richly formatted card.
|Function|Scenario|	Example|
|----|----|----|
|Action Commands|You want some external system to do an action and the result of the action to be sent back to your conversation.|Reserve a resource and allow the channel to know the reserved time slot.|
|Search Commands|You want to find something in an external system, and share the results with the conversation.|Search for a work item in Azure DevOps, and share it with the group as an Adaptive Card.|
|Link Unfurling|You want to complete a complex task involving multiple steps or lots of information in an external system, and share the results with a conversation.|Create a bug in your tracking system based on a Teams message, assign that bug to Bob, and send a card to the conversation thread with the bug's details.|

[Read More](https://learn.microsoft.com/en-us/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions?tabs=dotnet)

# SSO for Message Extension
There is [a MSDN document](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/authentication/bot-sso-code?tabs=cs1%2Ccs2%2Ccs3%2Ccs4&pivots=mex-app) going to introduce the SSO ability for Message Extension. This artile says that SSO handlers just work for `Search Actions` and `Link Unfurling`.

According to the mechanism introduced in the article, we have implemented an api `handleMessageExtensionQueryWithSSO` in `@microsoft/teamsfx`, which can help users realize me's sso more conveniently.

## API Introduce
Please read this API basic [introduce](https://github.com/OfficeDev/TeamsFx/blob/dev/docs/sdk/teamsfx.handlemessageextensionquerywithsso.md)    

## API Implement Sample
Please [reference this sample code](https://github.com/OfficeDev/TeamsFx-Samples/blob/49179ff8f766c2f99c5fa93d97ad8939ec880056/query-org-user-with-message-extension-sso/teamsBot.ts#L32) for the basic usage.

## API limitation
SSO only work for `query` and `link unfurling` in Message Extension. So this API should be implement within `TeamsActivityHandler.handleTeamsMessagingExtensionQuery`. 

In view of the poor SSO effect of link unfurling, the `API` has not been implemented for `TeamsActivityHandler.handleTeamsAppBasedLinkQuery` yet. If you have any idea about this, please submit an issue to us.