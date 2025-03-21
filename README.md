# Chess Match Tracking PowerApp

A PowerApp application for tracking chess matches between employees within the company.

## Overview

This application enables employees to log chess matches, track win/loss statistics, and maintain a history of games played. The app uses SharePoint lists to store match data and player statistics.

## Features

- Log new chess matches with details (players, date, result, time control, notes)
- Automatically track player statistics (wins, losses, draws)
- View match history
- Simple and intuitive interface

## Requirements

- Microsoft PowerApps license
- SharePoint Online
- Access to company Microsoft 365 environment

## Installation

1. Download the latest release MSAPP file
2. Open [PowerApps Studio](https://make.powerapps.com)
3. Click "Open" and select "Browse files"
4. Upload the MSAPP file
5. Set up the SharePoint lists as described in the documentation
6. Update data connections if necessary
7. Save and publish the app

## SharePoint Lists Setup

The app requires two SharePoint lists:

### ChessMatches
- ID (auto-generated)
- PlayerID (Single line of text)
- Player2ID (Single line of text)
- Date (Date and Time)
- Result (Choice: "Player 1 Win", "Player 2 Win", "Draw")
- TimeControl (Single line of text)
- Notes (Multiple lines of text)

### PlayerStats
- ID (auto-generated)
- PlayerID (Single line of text)
- Wins (Number)
- Losses (Number)
- Draws (Number)

## Contributing

1. Fork the repository
2. Create a new branch for your feature
3. Make your changes
4. Submit a pull request

## License

[LICENSE]

## Contact

(me@federicociro.com)[mailto:me@federicociro.com]
