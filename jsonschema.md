**Action messages JSON schema**
•	Establishes the format for client queries sent to the server.
•	Examples of actions are status, move, and stop.
{
  "type": "object",
  "properties": {
    "action": {
      "type": "string",
      "enum": ["move_to_pickup", "verify_item_position", "move_to_dropoff"]
    },
    "target": {
      "type": "string",
      "pattern": "^[A-Z]{3}_[A-Z]{2}_\\d{2}$"
    },
    "session_id": {
      "type": "string"
    },
    "coordinates": {
      "type": "array",
      "items": {
        "type": "number"
      },
      "minItems": 6,
      "maxItems": 6
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": ["action", "session_id", "timestamp"],
  "additionalProperties": false
}

Example
Pick up location
{
  "action": "move_to_pickup",
  "target": "MEL_RA_01",
  "session_id": "abcdef12345",
  "coordinates": [-90, 90, -100, 0, 0, -60],
  "timestamp": "2024-06-01T12:00:00Z"
}

Verify the pencil
{
  "action": "verify_item_position",
  "target": "MEL_RA_01",
  "session_id": "abcdef12345",
  "coordinates": [-84.81, 92.98, -99.22, 1.05, -0.87, -57.48],
  "timestamp": "2024-06-01T12:05:00Z",
  "status": "success",
  "message": "Pencil in position"
}
Drop off location
{
  "action": "move_to_dropoff",
  "target": "MEL_RA_01",
  "session_id": "abcdef12345",
  "coordinates": [90, 90, -100, 0, 0, -60],
  "timestamp": "2024-06-01T12:10:00Z"
}

**Session management JSON schema**
•	Manages the session states busy, timeout, and begin. 
•	Utilises unique tokens to guarantee accurate session monitoring. 
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "definitions": {
    "sessionToken": {
      "type": "string",
      "pattern": "^[a-fA-F0-9]{24}$"
    },
    "target": {
      "type": "string",
      "pattern": "^[A-Z]{3}_[A-Z]{2}_\\d{2}$"
    },
    "positionArray": {
      "type": "array",
      "items": {
        "type": "integer",
        "minimum": 0,
        "maximum": 180
      },
      "minItems": 6,
      "maxItems": 6
    },
    "statusEnum": {
      "type": "string",
      "enum": ["success", "error", "busy", "timeout", "session_begin"]
    }
 },
  "properties": {
    "status": {
      "$ref": "#/definitions/statusEnum"
    },
    "action": {
      "type": "string",
      "enum": ["move_to_pickup", "verify_item_position", "move_to_dropoff", "begin_session", "timeout_session"]
    },
    "session_token": {
      "$ref": "#/definitions/sessionToken"
    },
    "target": {
      "$ref": "#/definitions/target"
    },
    "message": {
      "type": "string"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "details": {
      "type": "object",
      "properties": {
        "position": {
          "$ref": "#/definitions/positionArray"
        },
        "gripper_state": {
          "type": "string",
          "enum": ["open", "closed"]
        },
        "error_code": {
          "type": "string"
        }
      }
    }
  },
  "required": ["status", "session_token", "target", "timestamp"]
}
Example
Begin message
{
  "status": "session_begin",
  "action": "begin_session",
  "session_token": "abcdef1234567890abcdef12",
  "target": "MEL_RA_01",
  "timestamp": "2024-06-01T12:00:00Z",
  "message": "Session started successfully."
}
Session timeout 
{
  "status": "timeout",
  "action": "timeout_session",
  "session_token": "abcdef1234567890abcdef12",
  "target": "MEL_RA_01",
  "timestamp": "2024-06-01T12:30:00Z",
  "message": "Session timed out due to inactivity."
}
Busy status
{
  "status": "busy",
  "session_token": "abcdef1234567890abcdef12",
  "target": "MEL_RA_01",
  "timestamp": "2024-06-01T12:05:00Z",
  "message": "MechArm is currently busy with another task."
}

**Response messages JSON schema**
•	Establishes the structure for the server's client responses. 
•	Comprises the timestamp, information, action, status, and message. 
{
"type": "object",
  "properties": {
    "response_type": {
      "type": "string",
      "enum": ["action_response", "session_response", "error_response"]
    },
    "status": {
      "type": "string",
      "enum": ["success", "error", "busy", "timeout"]
    },
    "session_id": {
      "type": "string"
    },
    "message": {
      "type": "string"
    },
    "data": {
      "type": "object",
      "properties": {
        "action": {
          "type": "string",
          "enum": ["move_to_pickup", "verify_item_position", "move_to_dropoff"]
        },
        "coordinates": {
          "type": "array",
          "items": {
            "type": "number"
          },
          "minItems": 6,
          "maxItems": 6
},
        "target": {
          "type": "string",
          "pattern": "^[A-Z]{3}_[A-Z]{2}_\\d{2}$"
        }
      }
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": ["response_type", "status", "session_id", "timestamp"],
  "additionalProperties": false
}
Example
Successful action
{
  "response_type": "action_response",
  "status": "success",
  "session_id": "abcdef12345",
  "message": "Move to pickup location completed.",
  "data": {
    "action": "move_to_pickup",
    "coordinates": [-90, 90, -100, 0, 0, -60],
    "target": "MEL_RA_01"
  },
  "timestamp": "2024-06-01T12:02:00Z"
}
Verification position
{
  "response_type": "action_response",
  "status": "success",
  "session_id": "abcdef12345",
  "message": "Pencil in position.",
  "data": {
    "action": "verify_item_position",
    "coordinates": [-84.81, 92.98, -99.22, 1.05, -0.87, -57.48],
    "target": "MEL_RA_01"
  },
  "timestamp": "2024-06-01T12:05:00Z"
}
Error response
{
  "response_type": "error_response",
  "status": "error",
  "session_id": "abcdef12345",
  "message": "Failed to move to dropoff location due to obstruction.",
  "timestamp": "2024-06-01T12:10:00Z"
}
