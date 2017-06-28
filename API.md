```
/// HTTP Request NOT INCLUDING THE BODY. This allows for streaming
public struct HTTPRequest {
    public var method : HTTPMethod
    public var target : String /* e.g. "/foo/bar?buz=qux" */
    public var httpVersion : HTTPVersion
    public var headers : HTTPHeaders
}

/// HTTP Response NOT INCLUDING THE BODY
public struct HTTPResponse {
    public var httpVersion : HTTPVersion
    public var status: HTTPResponseStatus
    public var transferEncoding: HTTPTransferEncoding
    public var headers: HTTPHeaders
}

/// Object that code writes the response and response body to. 
public protocol HTTPResponseWriter : class {
    func writeContinue(headers: HTTPHeaders?) /* to send an HTTP `100 Continue` */
    
    func writeResponse(_ response: HTTPResponse)
    
    func writeTrailer(key: String, value: String)
    
    func writeBody(data: DispatchData, completion: @escaping (Result<POSIXError, ()>) -> Void)
    func writeBody(data: DispatchData) /* convenience */

    func writeBody(data: Data, completion: @escaping (Result<POSIXError, ()>) -> Void)
    func writeBody(data: Data) /* convenience */

    func done() /* convenience */
    func done(completion: @escaping (Result<POSIXError, ()>) -> Void)
    func abort()
}

/// Method that takes a chunk of request body and is expected to write to the ResponseWriter
public typealias HTTPBodyHandler = (HTTPBodyChunk, inout Bool) -> Void /* the Bool can be set to true when we don't want to process anything further */

/// Indicates whether the body is going to be processed or ignored
public enum HTTPBodyProcessing {
    case discardBody /* if you're not interested in the body */
    case processBody(handler: HTTPBodyHandler)
}

/// Part (or maybe all) of the incoming request body
public enum HTTPBodyChunk {
    case chunk(data: DispatchData, finishedProcessing: () -> Void) /* a new bit of the HTTP request body has arrived, finishedProcessing() must be called when done with that chunk */
    case failed(error: /*HTTPParser*/ Error) /* error while streaming the HTTP request body, eg. connection closed */
    case trailer(key: String, value: String) /* trailer has arrived (this we actually haven't implemented yet) */
    case end /* body and trailers finished */
}

/// Headers structure.
public struct HTTPHeaders {
    var storage: [String:[String]]     /* lower cased keys */
    var original: [(String, String)]   /* original casing */
    let description: String
    
    public subscript(key: String) -> [String]
    func makeIterator() -> IndexingIterator<Array<(String, String)>>
    
    public init(_ headers: [(String, String)] = [])
}

/// Version number of the HTTP Protocol
public struct HTTPVersion {
    /// Major version component.
    public var major: Int
    /// Minor version component.
    public var minor: Int
    
    public init(major: Int, minor: Int)
}

public enum HTTPTransferEncoding {
    case identity(contentLength: UInt)
    case chunked
}

/// Response status (200 ok, 404 not found, etc)
public struct HTTPResponseStatus: Equatable, CustomStringConvertible, ExpressibleByIntegerLiteral {
    public let code: Int
    public let reasonPhrase: String

    public init(code: Int, reasonPhrase: String)
    public init(code: Int)
    
    /* all the codes from http://www.iana.org/assignments/http-status-codes */
    public static let `continue`
    public static let switchingProtocols
    public static let ok
    public static let created
    public static let accepted
    public static let nonAuthoritativeInformation
    public static let noContent
    public static let resetContent
    public static let partialContent
    public static let multiStatus
    public static let alreadyReported
    public static let imUsed
    public static let multipleChoices
    public static let movedPermanently
    public static let found
    public static let seeOther
    public static let notModified
    public static let useProxy
    public static let temporaryRedirect
    public static let permanentRedirect
    public static let badRequest
    public static let unauthorized
    public static let paymentRequired
    public static let forbidden
    public static let notFound
    public static let methodNotAllowed
    public static let notAcceptable
    public static let proxyAuthenticationRequired
    public static let requestTimeout
    public static let conflict
    public static let gone
    public static let lengthRequired
    public static let preconditionFailed
    public static let payloadTooLarge
    public static let uriTooLong
    public static let unsupportedMediaType
    public static let rangeNotSatisfiable
    public static let expectationFailed
    public static let misdirectedRequest
    public static let unprocessableEntity
    public static let locked
    public static let failedDependency
    public static let upgradeRequired
    public static let preconditionRequired
    public static let tooManyRequests
    public static let requestHeaderFieldsTooLarge
    public static let unavailableForLegalReasons
    public static let internalServerError
    public static let notImplemented
    public static let badGateway
    public static let serviceUnavailable
    public static let gatewayTimeout
    public static let httpVersionNotSupported
    public static let variantAlsoNegotiates
    public static let insufficientStorage
    public static let loopDetected
    public static let notExtended
    public static let networkAuthenticationRequired

    public var `class`: Class

    public enum Class {
        case informational
        case successful
        case redirection
        case clientError
        case serverError
        case invalidStatus
    }
}

/// HTTP Methods handled by http_parser.[ch] supports
public struct HTTPMethod : RawRepresentable {

    public let rawValue: String
    
    public init(rawValue: String)
    public init(_ rawValue: String)
        
    /* Constants for everything that http_parser.[ch] supports */
    public static let delete
    public static let get
    public static let head
    public static let post
    public static let put
    public static let connect
    public static let options
    public static let trace
    public static let copy
    public static let lock
    public static let mkol
    public static let move
    public static let propfind
    public static let proppatch
    public static let search
    public static let unlock
    public static let bind
    public static let rebind
    public static let unbind
    public static let acl
    public static let report
    public static let mkactivity
    public static let checkout
    public static let merge
    public static let msearch
    public static let notify
    public static let subscribe
    public static let unsubscribe
    public static let patch
    public static let purge
    public static let mkcalendar
    public static let link
    public static let unlink
}
```
